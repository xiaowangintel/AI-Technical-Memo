# grafana_push.py — Code Analysis / 代码分析
## Source / 来源
- **Path / 路径:** `test/benchmarks/utils/grafana_push.py`
- **EN:** Builds InfluxDB line-protocol payloads from benchmark records and posts them to an InfluxDB v2 endpoint for Grafana dashboards.
- **CN:** 把 benchmark 记录组装成 InfluxDB line protocol 负载，并发送到 InfluxDB v2 接口，供 Grafana 仪表盘使用。

## Line-by-Line Analysis / 逐行分析
### Lines 6-19
```python
import os
import time
import urllib.error
import urllib.request


def _escape_tag(value):
    return (
        str(value)
        .replace("\\", "\\\\")
        .replace(" ", "\\ ")
        .replace(",", "\\,")
        .replace("=", "\\=")
    )
```
**EN:** `_escape_tag()` converts any value to text and escapes backslashes, spaces, commas, and equal signs. Those characters have special meaning in InfluxDB line protocol tags, so this helper prevents malformed tag strings when commit ids or other identifiers are embedded.

**CN:** `_escape_tag()` 会把任意值转成字符串，并转义反斜杠、空格、逗号和等号。这些字符在 InfluxDB line protocol 的 tag 中具有特殊含义，因此该辅助函数可避免在嵌入提交号等标识符时生成非法 tag。

### Lines 22-66
```python
def _build_line_protocol(records, suite_name, timestamp_ns, git_commit_id=""):
    lines = []
    commit_tag = _escape_tag(git_commit_id) if git_commit_id else ""

    for record in records:
        kernel = (record.get("Kernel", ""))
 
        suite = (suite_name)
        result = record.get("Result", "")

        fields = []
        tflops = record.get("Tflops",0)
        throughput = record.get("Throughput",0)
        shape = record.get("Shape", "")
        kernel = kernel + f"_{shape}"

        if isinstance(shape, (list, tuple)):
            shape = "x".join(str(dim) for dim in shape)
        else:
            shape = str(shape)

        if tflops:
            fields.append(f"tflops={float(tflops)}")
        else:
            fields.append(f"tflops=0")
        if throughput:
            fields.append(f"throughput={float(throughput)}")
        else:
            fields.append(f"throughput=0")

        if result:
            fields.append(f"result=\"{str(result).replace('"', '\\"')}\"")

        if not fields:
            continue

        tag_set = f"suite={suite},kernel={kernel}"
        if commit_tag:
            fields.append(f"commit_id=\"{commit_tag}\"")

        line = f"cutlass_benchmarks,{tag_set} {','.join(fields)} {timestamp_ns}"

        lines.append(line)

    return "\n".join(lines) + "\n"
```
**EN:** `_build_line_protocol()` iterates over benchmark `records` and emits one measurement line per record. It reads `Kernel`, `Shape`, `Result`, `Tflops`, and `Throughput`, appends the shape text to the kernel name, normalizes tuple/list shapes into `x`-joined strings, and emits numeric fields for performance metrics plus string fields for result and optional commit id. The final measurement name is fixed as `cutlass_benchmarks`, with `suite` and `kernel` as tags and the caller-supplied nanosecond timestamp at the end.

**CN:** `_build_line_protocol()` 遍历 benchmark `records`，为每条记录生成一行 measurement。它读取 `Kernel`、`Shape`、`Result`、`Tflops` 和 `Throughput`，把形状文本拼接到 kernel 名称后面，把列表/元组形式的形状规范化为用 `x` 连接的字符串，并为性能指标输出数值 field，同时为结果和可选提交号输出字符串 field。最终 measurement 名固定为 `cutlass_benchmarks`，`suite` 与 `kernel` 作为 tag，结尾使用调用方提供的纳秒时间戳。

### Lines 69-108
```python
def push_results(records, suite_name, timestamp_ns=None, git_commit_id=""):
    influx_url = os.getenv("INFLUX_URL")
    influx_org = os.getenv("INFLUX_ORG")
    influx_bucket = os.getenv("INFLUX_BUCKET")
    influx_token = os.getenv("INFLUX_TOKEN")

    if not all([influx_url, influx_org, influx_bucket, influx_token]):
        print("InfluxDB config missing; skipping push (set INFLUX_URL/ORG/BUCKET/TOKEN).")
        return

    if timestamp_ns is None:
        timestamp_ns = int(time.time() * 1e9)

    payload = _build_line_protocol(records, suite_name, timestamp_ns, git_commit_id)
    print(payload)
    if not payload:
        print("No records to push to InfluxDB.")
        return

    write_url = (
        f"{influx_url.rstrip('/')}/api/v2/write"
        f"?org={influx_org}&bucket={influx_bucket}&precision=ns"
    )

    request = urllib.request.Request(write_url, data=payload.encode("utf-8"), method="POST")
    request.add_header("Authorization", f"Token {influx_token}")
    request.add_header("Content-Type", "text/plain; charset=utf-8")

    try:
        with urllib.request.urlopen(request) as response:
            if response.status >= 300:
                print(f"InfluxDB write failed: HTTP {response.status}")
            else:
                print("InfluxDB write succeeded.")
    except urllib.error.HTTPError as exc:
        error_body = exc.read().decode("utf-8") if exc.fp else ""
        print(f"InfluxDB write error: HTTP {exc.code} {error_body}")
    except urllib.error.URLError as exc:
        print(f"InfluxDB connection error: {exc.reason}")

```
**EN:** `push_results()` reads the InfluxDB connection settings from environment variables, skips uploading if any required variable is missing, and generates a default nanosecond timestamp when none is provided. It prints the payload for visibility, builds the `/api/v2/write` URL, sends a POST request with token authentication and `text/plain` content type, then reports success or surfaces HTTP/URL errors with readable messages.

**CN:** `push_results()` 从环境变量读取 InfluxDB 连接参数；如果缺少任意必需变量，就直接跳过上传；若未提供时间戳，则自动生成纳秒级时间戳。函数会先打印 payload 以便观察，再拼接 `/api/v2/write` URL，使用 Token 鉴权和 `text/plain` 内容类型发起 POST 请求，最后输出成功信息，或在 HTTP/URL 异常时给出可读错误消息。

## Key Concepts / 关键概念
- **EN:** InfluxDB line protocol encoding: measurement names, tags, fields, and timestamps are assembled manually.
  **CN:** InfluxDB line protocol 编码：measurement、tag、field 与时间戳由代码手工拼装。
- **EN:** Metric normalization: empty or missing throughput/TFLOPS values are converted to numeric zero fields.
  **CN:** 指标归一化：缺失或为空的吞吐量/TFLOPS 会被转换成数值 0。
- **EN:** Environment-driven configuration: upload behavior is entirely controlled by `INFLUX_*` variables.
  **CN:** 环境变量驱动配置：上传行为完全由 `INFLUX_*` 环境变量控制。

## Dependencies / 依赖关系
- **EN:** Standard library modules `os`, `time`, `urllib.request`, and `urllib.error` provide configuration lookup, timestamp generation, HTTP requests, and error handling.
  **CN:** 标准库模块 `os`、`time`、`urllib.request` 与 `urllib.error` 分别用于配置读取、时间戳生成、HTTP 请求与错误处理。
- **EN:** The function expects benchmark record dictionaries with keys such as `Kernel`, `Shape`, `Result`, `Tflops`, and `Throughput`—exactly the structure produced by `run_benchmarks.py`.
  **CN:** 该模块期望输入的 benchmark 记录字典包含 `Kernel`、`Shape`、`Result`、`Tflops`、`Throughput` 等键，这与 `run_benchmarks.py` 生成的结构完全对应。
- **EN:** External dependency: a reachable InfluxDB v2 service and valid `INFLUX_URL`, `INFLUX_ORG`, `INFLUX_BUCKET`, and `INFLUX_TOKEN` values.
  **CN:** 外部依赖：可访问的 InfluxDB v2 服务，以及有效的 `INFLUX_URL`、`INFLUX_ORG`、`INFLUX_BUCKET` 和 `INFLUX_TOKEN`。
