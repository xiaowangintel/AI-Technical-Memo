# FileUtil.h — Code Analysis / 代码分析

## Source / 来源
- File: `torch/nativert/common/FileUtil.h`
- Repository: `pytorch/pytorch`
- Purpose (EN): Declares Native Runtime common interfaces, helper types, and contracts for FileUtil.
- 用途 (CN): 声明 Native Runtime 中 common 子模块里与 FileUtil 相关的接口、辅助类型和约定。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```cpp
#pragma once

/*
 * Ported from folly/FileUtil.h
 */
#include <limits>
#include <string_view>
```
- EN: Documents intent, assumptions, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、假设或兼容性约束。

### Lines 8-15
```cpp

#include <fcntl.h>
#include <sys/stat.h>
#include <sys/types.h>

// Copied from folly/portability/SysTypes.h
#ifdef _WIN32
#include <basetsd.h>
```
- EN: Documents intent, assumptions, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、假设或兼容性约束。

### Lines 16-24
```cpp

// This is a massive pain to have be an `int` due to the pthread implementation
// we support, but it's far more compatible with the rest of the windows world
// as an `int` than it would be as a `void*`
using pid_t = int;

using uid_t = int;
using gid_t = int;

```
- EN: This block protects shared state or ordering guarantees. Key symbols: `pid_t`, `uid_t`, `gid_t`.
- CN: 该代码块保护共享状态或执行顺序保证。关键符号：`pid_t`, `uid_t`, `gid_t`。

### Lines 25-36
```cpp
// This isn't actually supposed to be defined here, but it's the most
// appropriate place without defining a portability header for stdint.h
// with just this single typedef.
using ssize_t = SSIZE_T;

#ifndef HAVE_MODE_T
#define HAVE_MODE_T 1
// The Windows headers don't define this anywhere, nor do any of the libs
// that Folly depends on, so define it here.
using mode_t = unsigned int;
#endif

```
- EN: This block iterates over collections or execution units. Key symbols: `ssize_t`, `mode_t`.
- CN: 该代码块遍历集合或执行单元。关键符号：`ssize_t`, `mode_t`。

### Lines 37-43
```cpp
// Copied from folly/portability/Fcntl.h
#define O_CLOEXEC _O_NOINHERIT
#endif

#include <c10/util/Exception.h>
#include <c10/util/ScopeExit.h>

```
- EN: Documents intent, assumptions, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、假设或兼容性约束。

### Lines 44-51
```cpp
namespace torch::nativert {
class File {
 public:
  /**
   * Creates an empty File object, for late initialization.
   */
  constexpr File() noexcept : fd_(-1), ownsFd_(false) {}

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `File`, `fd_`, `ownsFd_`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`File`, `fd_`, `ownsFd_`。

### Lines 52-59
```cpp
  /**
   * Create a File object from an existing file descriptor.
   *
   * @param fd Existing file descriptor
   * @param ownsFd Takes ownership of the file descriptor if ownsFd is true.
   */
  explicit File(int fd, bool ownsFd = false) noexcept;

```
- EN: This block handles conditional control flow. Key symbols: `File`.
- CN: 该代码块处理条件控制流。关键符号：`File`。

### Lines 60-68
```cpp
  /**
   * Open and create a file object.  Throws on error.
   * Owns the file descriptor implicitly.
   */
  explicit File(
      std::string_view name,
      int flags = O_RDONLY,
      mode_t mode = 0666);

```
- EN: This block reports or normalizes error conditions. Key symbols: `File`.
- CN: 该代码块报告或规范化错误情况。关键符号：`File`。

### Lines 69-75
```cpp
  ~File();

  /**
   * Create and return a temporary, owned file (uses tmpfile()).
   */
  static File temporary();

```
- EN: This block returns results to callers or downstream stages. Key symbols: `~File`, `file`, `tmpfile`, `temporary`.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：`~File`, `file`, `tmpfile`, `temporary`。

### Lines 76-82
```cpp
  /**
   * Return the file descriptor, or -1 if the file was closed.
   */
  int fd() const {
    return fd_;
  }

```
- EN: This block handles conditional control flow; returns results to callers or downstream stages. Key symbols: `fd`.
- CN: 该代码块处理条件控制流；向调用方或后续阶段返回结果。关键符号：`fd`。

### Lines 83-89
```cpp
  /**
   * Returns 'true' iff the file was successfully opened.
   */
  explicit operator bool() const {
    return fd_ != -1;
  }

```
- EN: This block returns results to callers or downstream stages. Key symbols: `bool`.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：`bool`。

### Lines 90-100
```cpp
  /**
   * If we own the file descriptor, close the file and throw on error.
   * Otherwise, do nothing.
   */
  void close();

  /**
   * Closes the file (if owned).  Returns true on success, false (and sets
   * errno) on error.
   */
  bool closeNoThrow();
```
- EN: This block reports or normalizes error conditions; handles conditional control flow. Key symbols: `close`, `file`, `false`, `closeNoThrow`.
- CN: 该代码块报告或规范化错误情况；处理条件控制流。关键符号：`close`, `file`, `false`, `closeNoThrow`。

### Lines 101-107
```cpp

  /**
   * Returns and releases the file descriptor; no longer owned by this File.
   * Returns -1 if the File object didn't wrap a file.
   */
  int release() noexcept;

```
- EN: This block handles conditional control flow. Key symbols: `release`.
- CN: 该代码块处理条件控制流。关键符号：`release`。

### Lines 108-116
```cpp
  /**
   * Swap this File with another.
   */
  void swap(File& other) noexcept;

  // movable
  File(File&& /*other*/) noexcept;
  File& operator=(File&& /*other*/) noexcept;

```
- EN: This block implements local helper logic for FileUtil. Key symbols: `swap`, `File`.
- CN: 该代码块实现与 FileUtil 相关的局部辅助逻辑。关键符号：`swap`, `File`。

### Lines 117-123
```cpp
 private:
  // unique
  File(const File&) = delete;
  File& operator=(const File&) = delete;

  int fd_;
  bool ownsFd_;
```
- EN: This block implements local helper logic for FileUtil. Key symbols: `File`.
- CN: 该代码块实现与 FileUtil 相关的局部辅助逻辑。关键符号：`File`。

### Lines 124-134
```cpp
};

/**
 * Convenience wrappers around some commonly used system calls.  The *NoInt
 * wrappers retry on EINTR.  The *Full wrappers retry on EINTR and also loop
 * until all data is written.  Note that *Full wrappers weaken the thread
 * semantics of underlying system calls.
 */
int openNoInt(const char* name, int flags, mode_t mode = 0666);
int closeNoInt(int fd);

```
- EN: This block protects shared state or ordering guarantees. Key symbols: `openNoInt`, `closeNoInt`.
- CN: 该代码块保护共享状态或执行顺序保证。关键符号：`openNoInt`, `closeNoInt`。

### Lines 135-148
```cpp
/**
 * Similar to readFull and preadFull above, wrappers around write() and
 * pwrite() that loop until all data is written.
 *
 * Generally, the write() / pwrite() system call may always write fewer bytes
 * than requested, just like read().  In certain cases (such as when writing to
 * a pipe), POSIX provides stronger guarantees, but not in the general case.
 * For example, Linux (even on a 64-bit platform) won't write more than 2GB in
 * one write() system call.
 *
 * Note that writevFull and pwritevFull require iov to be non-const, unlike
 * writev and pwritev.  The contents of iov after these functions return
 * is unspecified.
 *
```
- EN: Documents intent, assumptions, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、假设或兼容性约束。

### Lines 149-162
```cpp
 * These functions return -1 on error, or the total number of bytes written
 * (which is always the same as the number of requested bytes) on success.
 */
ssize_t writeFull(int fd, const void* buf, size_t count);

/**
 * Wrapper around read() (and pread()) that, in addition to retrying on
 * EINTR, will loop until all data is read.
 *
 * This wrapper is only useful for blocking file descriptors (for non-blocking
 * file descriptors, you have to be prepared to deal with incomplete reads
 * anyway), and only exists because POSIX allows read() to return an incomplete
 * read if interrupted by a signal (instead of returning -1 and setting errno
 * to EINTR).
```
- EN: This block reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages; protects shared state or ordering guarantees. Key symbols: `writeFull`, `read`, `pread`, `descriptors`, `signal`.
- CN: 该代码块报告或规范化错误情况；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果；保护共享状态或执行顺序保证。关键符号：`writeFull`, `read`, `pread`, `descriptors`, `signal`。

### Lines 163-176
```cpp
 *
 * Note that this wrapper weakens the thread safety of read(): the file pointer
 * is shared between threads, but the system call is atomic.  If multiple
 * threads are reading from a file at the same time, you don't know where your
 * data came from in the file, but you do know that the returned bytes were
 * contiguous.  You can no longer make this assumption if using readFull().
 * You should probably use pread() when reading from the same file descriptor
 * from multiple threads simultaneously, anyway.
 *
 * Note that readvFull and preadvFull require iov to be non-const, unlike
 * readv and preadv.  The contents of iov after these functions return
 * is unspecified.
 */
[[nodiscard]] ssize_t readFull(int fd, void* buf, size_t count);
```
- EN: This block handles conditional control flow; protects shared state or ordering guarantees. Key symbols: `read`, `readFull`, `pread`.
- CN: 该代码块处理条件控制流；保护共享状态或执行顺序保证。关键符号：`read`, `readFull`, `pread`。

### Lines 177-188
```cpp

/**
 * Read entire file (if num_bytes is defaulted) or no more than
 * num_bytes (otherwise) into container *out. The container is assumed
 * to be contiguous, with element size equal to 1, and offer size(),
 * reserve(), and random access (e.g. std::vector<char>, std::string,
 * fbstring).
 *
 * Returns: true on success or false on failure. In the latter case
 * errno will be set appropriately by the failing system primitive.
 */
template <class Container>
```
- EN: Declares C++ types or containers needed in this file. Representative symbols: `file`, `num_bytes`, `size`, `reserve`, `access`, `Container`.
- CN: 声明该文件需要的 C++ 类型或容器。代表性符号：`file`, `num_bytes`, `size`, `reserve`, `access`, `Container`。

### Lines 189-196
```cpp
bool readFile(
    int fd,
    Container& out,
    size_t num_bytes = std::numeric_limits<size_t>::max()) {
  static_assert(
      sizeof(out[0]) == 1,
      "readFile: only containers with byte-sized elements accepted");

```
- EN: This block checks invariants or expected outcomes. Key symbols: `readFile`, `max`, `static_assert`.
- CN: 该代码块检查不变量或预期结果。关键符号：`readFile`, `max`, `static_assert`。

### Lines 197-204
```cpp
  size_t soFar = 0; // amount of bytes successfully read
  auto guard = c10::make_scope_exit([&]() {
    assert(out.size() >= soFar); // resize better doesn't throw
    out.resize(soFar);
  });

  // Obtain file size:
  struct stat buf;
```
- EN: Declares C++ types or containers needed in this file. Representative symbols: `make_scope_exit`, `assert`, `size`, `resize`, `stat`.
- CN: 声明该文件需要的 C++ 类型或容器。代表性符号：`make_scope_exit`, `assert`, `size`, `resize`, `stat`。

### Lines 205-216
```cpp
  if (fstat(fd, &buf) == -1) {
    return false;
  }
  // Some files (notably under /proc and /sys on Linux) lie about
  // their size, so treat the size advertised by fstat under advise
  // but don't rely on it. In particular, if the size is zero, we
  // should attempt to read stuff. If not zero, we'll attempt to read
  // one extra byte.
  constexpr size_t initialAlloc = 1024 * 4;
  out.resize(std::min(
      buf.st_size > 0 ? (size_t(buf.st_size) + 1) : initialAlloc, num_bytes));

```
- EN: This block handles conditional control flow; returns results to callers or downstream stages. Key symbols: `fstat`, `resize`, `min`, `size_t`.
- CN: 该代码块处理条件控制流；向调用方或后续阶段返回结果。关键符号：`fstat`, `resize`, `min`, `size_t`。

### Lines 217-223
```cpp
  while (soFar < out.size()) {
    const auto actual = readFull(fd, &out[soFar], out.size() - soFar);
    if (actual == -1) {
      return false;
    }
    soFar += actual;
    if (soFar < out.size()) {
```
- EN: This block handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: `size`, `readFull`.
- CN: 该代码块处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：`size`, `readFull`。

### Lines 224-231
```cpp
      // File exhausted
      break;
    }
    // Ew, allocate more memory. Use exponential growth to avoid
    // quadratic behavior. Cap size to num_bytes.
    out.resize(std::min(out.size() * 3 / 2, num_bytes));
  }

```
- EN: This block reuses computed state to reduce repeated work. Key symbols: `resize`, `min`, `size`.
- CN: 该代码块复用已计算状态以减少重复工作。关键符号：`resize`, `min`, `size`。

### Lines 232-238
```cpp
  return true;
}

/**
 * Same as above, but takes in a file name instead of fd
 */
template <class Container>
```
- EN: Declares C++ types or containers needed in this file. Representative symbols: `Container`.
- CN: 声明该文件需要的 C++ 类型或容器。代表性符号：`Container`。

### Lines 239-246
```cpp
bool readFile(
    const char* file_name,
    Container& out,
    size_t num_bytes = std::numeric_limits<size_t>::max()) {
  TORCH_CHECK(file_name);

  const auto fd = openNoInt(file_name, O_RDONLY | O_CLOEXEC);
  if (fd == -1) {
```
- EN: This block checks invariants or expected outcomes; handles conditional control flow. Key symbols: `readFile`, `max`, `openNoInt`.
- CN: 该代码块检查不变量或预期结果；处理条件控制流。关键符号：`readFile`, `max`, `openNoInt`。

### Lines 247-254
```cpp
    return false;
  }

  auto guard = c10::make_scope_exit([&]() {
    // Ignore errors when closing the file
    closeNoInt(fd);
  });

```
- EN: This block reports or normalizes error conditions; returns results to callers or downstream stages; protects shared state or ordering guarantees. Key symbols: `make_scope_exit`, `closeNoInt`.
- CN: 该代码块报告或规范化错误情况；向调用方或后续阶段返回结果；保护共享状态或执行顺序保证。关键符号：`make_scope_exit`, `closeNoInt`。

### Lines 255-258
```cpp
  return readFile(fd, out, num_bytes);
}

} // namespace torch::nativert
```
- EN: This block returns results to callers or downstream stages. Key symbols: `readFile`.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：`readFile`。


## Key Concepts / 关键概念
- EN: Testing assertions — the file focuses on checking values, errors, or textual expectations.
  CN: 测试断言——该文件重点检查数值、异常或文本期望是否满足要求。
- EN: Concurrency support — the implementation manages coordination, ownership, or thread safety.
  CN: 并发支持——实现会管理协作、所有权或线程安全。
- EN: Runtime composition — the file participates in assembling native runtime data flow, execution state, or backend handoff.
  CN: 运行时组合——该文件参与组织原生运行时的数据流、执行状态或后端交接。
- EN: Header-level integration — the translation unit depends on nearby runtime and framework declarations.
  CN: 头文件级集成——该翻译单元依赖周边运行时与框架声明。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `c10/util/Exception.h`, `c10/util/ScopeExit.h`
- External includes / 外部头文件: `limits`, `string_view`, `fcntl.h`, `sys/stat.h`, `sys/types.h`, `basetsd.h`
- Namespaces / 命名空间: `torch::nativert`
- Representative symbols / 代表性符号: `pid_t`, `uid_t`, `gid_t`, `ssize_t`, `mode_t`, `File`, `fd_`, `ownsFd_`, `~File`, `file`, `...`
