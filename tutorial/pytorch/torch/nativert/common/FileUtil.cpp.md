# FileUtil.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/nativert/common/FileUtil.cpp`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements Native Runtime common behavior for FileUtil, including execution, delegation, or low-level utilities.
- 用途 (CN): 实现 Native Runtime 中 common 子模块里与 FileUtil 相关的行为，包括执行、delegate 协调或底层工具逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11
```cpp
#include <torch/nativert/common/FileUtil.h>

#ifdef _WIN32
#include <io.h>
#define open _open
#define read _read
#define write _write
#define fileno _fileno
#define dup _dup
#else
#include <unistd.h>
```
- EN: Pulls in compile-time configuration or headers required here. Internal includes: `torch/nativert/common/FileUtil.h`; external includes: `io.h`, `unistd.h`.
- CN: 引入此处所需的编译期开关或头文件。内部依赖：`torch/nativert/common/FileUtil.h`；外部依赖：`io.h`, `unistd.h`。

### Lines 12-18
```cpp
#endif
#include <cerrno>

#include <c10/util/Exception.h>
#include <fmt/core.h>

namespace torch::nativert {
```
- EN: This block reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 19-26
```cpp

namespace {

int unistd_close(int fh) {
#ifdef _WIN32
  return ::_close(fh);
#else
  return ::close(fh);
```
- EN: This block returns results to callers or downstream stages. Key symbols: `unistd_close`, `_close`, `close`.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：`unistd_close`, `_close`, `close`。

### Lines 27-35
```cpp
#endif
}

inline void incr(ssize_t /*unused*/) {}
template <typename Offset>
inline void incr(ssize_t n, Offset& offset) {
  offset += static_cast<Offset>(n);
}

```
- EN: This block implements local helper logic for FileUtil. Key symbols: `incr`.
- CN: 该代码块实现与 FileUtil 相关的局部辅助逻辑。关键符号：`incr`。

### Lines 36-48
```cpp
// Wrap call to read/pread/write/pwrite(fd, buf, count, offset?) to retry on
// incomplete reads / writes.  The variadic argument magic is there to support
// an additional argument (offset) for pread / pwrite; see the incr() functions
// above which do nothing if the offset is not present and increment it if it
// is.
template <class F, class... Offset>
ssize_t wrapFull(F f, int fd, void* buf, size_t count, Offset... offset) {
  char* b = static_cast<char*>(buf);
  ssize_t totalBytes = 0;
  ssize_t r = -1;
  do {
    r = f(fd, b, count, offset...);
    if (r == -1) {
```
- EN: Declares C++ types or containers needed in this file. Representative symbols: `F`, `wrapFull`, `f`.
- CN: 声明该文件需要的 C++ 类型或容器。代表性符号：`F`, `wrapFull`, `f`。

### Lines 49-60
```cpp
      if (errno == EINTR) {
        continue;
      }
      return r;
    }

    totalBytes += r;
    b += r;
    count -= r;
    incr(r, offset...);
  } while (r != 0 && count); // 0 means EOF

```
- EN: This block handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: `incr`.
- CN: 该代码块处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：`incr`。

### Lines 61-74
```cpp
  return totalBytes;
}

int filterCloseReturn(int r) {
  // Ignore EINTR.  On Linux, close() may only return EINTR after the file
  // descriptor has been closed, so you must not retry close() on EINTR --
  // in the best case, you'll get EBADF, and in the worst case, you'll end up
  // closing a different file (one opened from another thread).
  //
  // Interestingly enough, the Single Unix Specification says that the state
  // of the file descriptor is unspecified if close returns EINTR.  In that
  // case, the safe thing to do is also not to retry close() -- leaking a file
  // descriptor is definitely better than closing the wrong file.
  if (r == -1 && errno == EINTR) {
```
- EN: This block handles conditional control flow; returns results to callers or downstream stages; protects shared state or ordering guarantees. Key symbols: `filterCloseReturn`.
- CN: 该代码块处理条件控制流；向调用方或后续阶段返回结果；保护共享状态或执行顺序保证。关键符号：`filterCloseReturn`。

### Lines 75-82
```cpp
    return 0;
  }
  return r;
}

//  The following wrapX() functions are private functions for wrapping file-io
//  against interrupt and partial op completions.

```
- EN: This block iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 83-90
```cpp
// Wrap call to f(args) in loop to retry on EINTR
template <class F, class... Args>
ssize_t wrapNoInt(F f, Args... args) {
  ssize_t r = -1;
  do {
    r = f(std::forward<Args>(args)...);
  } while (r == -1 && errno == EINTR);
  return r;
```
- EN: Declares C++ types or containers needed in this file. Representative symbols: `F`, `wrapNoInt`, `f`.
- CN: 声明该文件需要的 C++ 类型或容器。代表性符号：`F`, `wrapNoInt`, `f`。

### Lines 91-104
```cpp
}

} // namespace

int openNoInt(const char* name, int flags, mode_t mode) {
  // Android NDK bionic with FORTIFY has this definition:
  // https://android.googlesource.com/platform/bionic/+/9349b9e51b/libc/include/bits/fortify/fcntl.h
  // ```
  // __BIONIC_ERROR_FUNCTION_VISIBILITY
  // int open(const char* pathname, int flags, mode_t modes, ...) __overloadable
  //         __errorattr(__open_too_many_args_error);
  // ```
  // This is originally to prevent open() with incorrect parameters.
  //
```
- EN: This block reports or normalizes error conditions. Key symbols: `openNoInt`.
- CN: 该代码块报告或规范化错误情况。关键符号：`openNoInt`。

### Lines 105-111
```cpp
  // However, combined with folly wrapNotInt, template deduction will fail.
  // In this case, we create a custom lambda to bypass the error.
  // The solution is referenced from
  // https://github.com/llvm/llvm-project/commit/0a0e411204a2baa520fd73a8d69b664f98b428ba
  //
  auto openWrapper = [&] { return open(name, flags, mode); };
  return int(wrapNoInt(openWrapper));
```
- EN: This block reports or normalizes error conditions; returns results to callers or downstream stages. Key symbols: `open`, `int`, `wrapNoInt`.
- CN: 该代码块报告或规范化错误情况；向调用方或后续阶段返回结果。关键符号：`open`, `int`, `wrapNoInt`。

### Lines 112-120
```cpp
}

int closeNoInt(int fd) {
  return filterCloseReturn(unistd_close(fd));
}

ssize_t writeFull(int fd, const void* buf, size_t count) {
  // NOLINTNEXTLINE(cppcoreguidelines-pro-type-const-cast)
  return wrapFull(write, fd, const_cast<void*>(buf), count);
```
- EN: This block returns results to callers or downstream stages. Key symbols: `closeNoInt`, `filterCloseReturn`, `unistd_close`, `writeFull`, `wrapFull`.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：`closeNoInt`, `filterCloseReturn`, `unistd_close`, `writeFull`, `wrapFull`。

### Lines 121-131
```cpp
}

ssize_t readFull(int fd, void* buf, size_t count) {
  return wrapFull(read, fd, buf, count);
}

File::File(int fd, bool ownsFd) noexcept : fd_(fd), ownsFd_(ownsFd) {
  TORCH_CHECK(fd >= -1, "fd must be -1 or non-negative");
  TORCH_CHECK(fd != -1 || !ownsFd, "cannot own -1");
}

```
- EN: This block checks invariants or expected outcomes; returns results to callers or downstream stages. Key symbols: `readFull`, `wrapFull`, `File`, `fd_`, `ownsFd_`.
- CN: 该代码块检查不变量或预期结果；向调用方或后续阶段返回结果。关键符号：`readFull`, `wrapFull`, `File`, `fd_`, `ownsFd_`。

### Lines 132-145
```cpp
File::File(std::string_view name, int flags, mode_t mode)
    : fd_(::open(std::string(name).c_str(), flags, mode)), ownsFd_(false) {
  TORCH_CHECK(
      fd_ != 1,
      "open(\"",
      name,
      "\", ",
      flags,
      ", 0",
      mode,
      ") returned stdout.")
  ownsFd_ = true;
}

```
- EN: This block checks invariants or expected outcomes. Key symbols: `File`, `fd_`, `open`, `string`, `c_str`, `ownsFd_`.
- CN: 该代码块检查不变量或预期结果。关键符号：`File`, `fd_`, `open`, `string`, `c_str`, `ownsFd_`。

### Lines 146-153
```cpp
File::File(File&& other) noexcept : fd_(other.fd_), ownsFd_(other.ownsFd_) {
  other.release();
}

File& File::operator=(File&& other) noexcept {
  closeNoThrow();
  swap(other);
  return *this;
```
- EN: This block returns results to callers or downstream stages. Key symbols: `File`, `fd_`, `ownsFd_`, `release`, `closeNoThrow`, `swap`.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：`File`, `fd_`, `ownsFd_`, `release`, `closeNoThrow`, `swap`。

### Lines 154-167
```cpp
}

File::~File() {
  auto fd = fd_;
  if (!closeNoThrow()) { // ignore most errors
    TORCH_CHECK(
        errno != EBADF,
        "closing fd ",
        fd,
        ", it may already ",
        "have been closed. Another time, this might close the wrong FD.");
  }
}

```
- EN: This block checks invariants or expected outcomes; reports or normalizes error conditions; handles conditional control flow. Key symbols: `~File`, `closeNoThrow`.
- CN: 该代码块检查不变量或预期结果；报告或规范化错误情况；处理条件控制流。关键符号：`~File`, `closeNoThrow`。

### Lines 168-176
```cpp
/* static */ File File::temporary() {
  // make a temp file with tmpfile(), dup the fd, then return it in a File.
  FILE* tmpFile = tmpfile();
  TORCH_CHECK(tmpFile != nullptr, "tmpfile() failed");
  auto guard = c10::make_scope_exit([&]() { fclose(tmpFile); });

  int fd = ::dup(fileno(tmpFile));
  TORCH_CHECK(fd != -1, "dup() failed");

```
- EN: This block checks invariants or expected outcomes; returns results to callers or downstream stages; protects shared state or ordering guarantees. Key symbols: `temporary`, `tmpfile`, `make_scope_exit`, `fclose`, `dup`, `fileno`.
- CN: 该代码块检查不变量或预期结果；向调用方或后续阶段返回结果；保护共享状态或执行顺序保证。关键符号：`temporary`, `tmpfile`, `make_scope_exit`, `fclose`, `dup`, `fileno`。

### Lines 177-184
```cpp
  return File(fd, true);
}

int File::release() noexcept {
  int released = fd_;
  fd_ = -1;
  ownsFd_ = false;
  return released;
```
- EN: This block returns results to callers or downstream stages. Key symbols: `File`, `release`.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：`File`, `release`。

### Lines 185-192
```cpp
}

void File::swap(File& other) noexcept {
  using std::swap;
  swap(fd_, other.fd_);
  swap(ownsFd_, other.ownsFd_);
}

```
- EN: This block implements local helper logic for FileUtil. Key symbols: `swap`, `std`.
- CN: 该代码块实现与 FileUtil 相关的局部辅助逻辑。关键符号：`swap`, `std`。

### Lines 193-200
```cpp
void File::close() {
  TORCH_CHECK(closeNoThrow(), "close() failed");
}

[[nodiscard]] bool File::closeNoThrow() {
  int r = ownsFd_ ? unistd_close(fd_) : 0;
  release();
  return r == 0;
```
- EN: This block checks invariants or expected outcomes; returns results to callers or downstream stages. Key symbols: `close`, `closeNoThrow`, `unistd_close`, `release`.
- CN: 该代码块检查不变量或预期结果；向调用方或后续阶段返回结果。关键符号：`close`, `closeNoThrow`, `unistd_close`, `release`。

### Lines 201-203
```cpp
}

} // namespace torch::nativert
```
- EN: This block implements local helper logic for FileUtil. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 FileUtil 相关的局部辅助逻辑。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Concurrency support — the implementation manages coordination, ownership, or thread safety.
  CN: 并发支持——实现会管理协作、所有权或线程安全。
- EN: Runtime composition — the file participates in assembling native runtime data flow, execution state, or backend handoff.
  CN: 运行时组合——该文件参与组织原生运行时的数据流、执行状态或后端交接。
- EN: Header-level integration — the translation unit depends on nearby runtime and framework declarations.
  CN: 头文件级集成——该翻译单元依赖周边运行时与框架声明。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/nativert/common/FileUtil.h`, `c10/util/Exception.h`
- External includes / 外部头文件: `io.h`, `unistd.h`, `cerrno`, `fmt/core.h`
- Namespaces / 命名空间: `torch::nativert`
- Representative symbols / 代表性符号: `unistd_close`, `_close`, `close`, `incr`, `F`, `wrapFull`, `f`, `filterCloseReturn`, `wrapNoInt`, `openNoInt`, `...`
