# runner_utilization_report.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/ci/utils/runner_utilization_report.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `runner_utilization_report` workflow in SGLang. It mainly handles CI orchestration, test execution. / 该Python 模块用于支撑 SGLang 中的 `runner_utilization_report` 流程，主要负责CI 编排、测试执行。它属于 `utils` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: File header and usage notes / 文件头与使用说明
```python
#!/usr/bin/env python3
```
**EN:** These opening lines establish the file context, such as the shebang, encoding note, or top-level comments that tell operators how the script should be used.
**CN:** 这些起始行用于建立文件上下文，例如 shebang、编码说明或顶层注释，帮助使用者理解脚本的执行方式。

### Lines 2-7: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""
Runner Utilization Report

Analyzes GitHub Actions job data to calculate runner utilization metrics.
Reports idle time, active time, and utilization percentage per runner label.
"""
```
**EN:** Runner Utilization Report Analyzes GitHub Actions job data to calculate runner utilization metrics.
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 9-17: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import argparse
import json
import os
import random
import subprocess
import time
from collections import defaultdict
from concurrent.futures import ThreadPoolExecutor, as_completed
from datetime import datetime, timedelta, timezone
```
**EN:** This block loads argparse, json, os, random, subprocess, time, collections, concurrent.futures. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 argparse, json, os, random, subprocess, time, collections, concurrent.futures。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 20-21: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
DEFAULT_LABELS_TO_IGNORE = {"self-hosted", "Linux", "X64", "ARM64"}
GITHUB_HOSTED_LABELS = {"ubuntu-latest", "ubuntu-22.04", "ubuntu-24.04"}
```
**EN:** This section defines DEFAULT_LABELS_TO_IGNORE, GITHUB_HOSTED_LABELS, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 DEFAULT_LABELS_TO_IGNORE, GITHUB_HOSTED_LABELS，用于把可复用的默认值集中在模块顶部。

### Lines 24-68: Defines the `run_gh_command` routine / 定义 `run_gh_command` 例程
```python
def run_gh_command(args: list[str], max_retries: int = 10) -> dict:
    """Run gh CLI command and return JSON result.

    Retries on transient failures (5xx, secondary rate limits, network
    blips) with exponential backoff. The previous fail-fast behavior
    combined with `except Exception: return None` in the threadpool
    callers caused entire workflow runs to be silently dropped from
    the utilization numerator whenever GH API hiccuped, severely
    undercounting busy time on busy days.
    """
    last_err = ""
    for attempt in range(max_retries):
        result = subprocess.run(
            ["gh", "api"] + args,
            capture_output=True,
            text=True,
        )
        if result.returncode == 0:
            return json.loads(result.stdout)
        last_err = result.stderr or "(no stderr)"
        # Detect retryable conditions: HTTP 5xx, secondary rate limit, abuse
        # detection, network resets. 4xx other than 429 are non-retryable.
        retryable = any(
            s in last_err
            for s in (
                "rate limit",
                "abuse",
                "Internal Server Error",
                "502",
                "503",
                "504",
                "Bad Gateway",
                "Gateway Time-out",
                "connection reset",
                "Connection reset",
                "EOF",
                "timeout",
            )
        )
        if not retryable:
            break
        # Exponential backoff with jitter, capped at 60s.
        delay = min(60, (2**attempt) + random.uniform(0, 1))
        time.sleep(delay)
    raise Exception(f"gh api failed after {max_retries} attempts: {last_err[:300]}")
```
**EN:** This block defines `run_gh_command`. Run gh CLI command and return JSON result. It accepts 2 parameter(s): args, max_retries. Internally it loads structured data, executes external commands.
**CN:** 该代码块定义了 `run_gh_command`。其文档字符串说明了该逻辑的预期职责。它接收 2 个参数：args, max_retries。内部会加载结构化数据、执行外部命令。

### Lines 71-99: Defines the `get_workflow_runs` routine / 定义 `get_workflow_runs` 例程
```python
def get_workflow_runs(repo: str, hours: int = 24) -> list[dict]:
    """Get workflow runs from the last N hours."""
    since = datetime.now(timezone.utc) - timedelta(hours=hours)

    runs = []
    page = 1
    while True:
        data = run_gh_command(
            [
                f"repos/{repo}/actions/runs?per_page=100&page={page}",
            ]
        )
        page_runs = data.get("workflow_runs", [])

        # Filter by time
        for run in page_runs:
            created_at = parse_time(run.get("created_at"))
            if created_at and created_at >= since:
                runs.append(run)
            elif created_at and created_at < since:
                # Runs are ordered by created_at desc, so we can stop
                return runs

        if len(page_runs) < 100:
            break
        page += 1
        if page > 50:  # Safety limit (5000 runs)
            break
    return runs
```
**EN:** This block defines `get_workflow_runs`. Get workflow runs from the last N hours. It accepts 2 parameter(s): repo, hours. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `get_workflow_runs`。其文档字符串说明了该逻辑的预期职责。它接收 2 个参数：repo, hours。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 102-126: Defines the `get_jobs_for_run` routine / 定义 `get_jobs_for_run` 例程
```python
def get_jobs_for_run(repo: str, run_id: int) -> list[dict]:
    """Get all jobs for a workflow run, including all retry attempts.

    `filter=all` is required so that re-run attempts of the same job
    appear separately. Each attempt consumed host time on the runner
    pool, so for utilization we want them all summed in. The default
    (`filter=latest`) only returns the most recent attempt and silently
    hides time spent on prior retries.
    """
    jobs = []
    page = 1
    while True:
        data = run_gh_command(
            [
                f"repos/{repo}/actions/runs/{run_id}/jobs"
                f"?per_page=100&page={page}&filter=all",
            ]
        )
        jobs.extend(data.get("jobs", []))
        if len(data.get("jobs", [])) < 100:
            break
        page += 1
        if page > 20:  # Safety limit (2000 jobs per run)
            break
    return jobs
```
**EN:** This block defines `get_jobs_for_run`. Get all jobs for a workflow run, including all retry attempts. It accepts 2 parameter(s): repo, run_id. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `get_jobs_for_run`。其文档字符串说明了该逻辑的预期职责。它接收 2 个参数：repo, run_id。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 129-150: Defines the `get_runners` routine / 定义 `get_runners` 例程
```python
def get_runners(repo: str, online_only: bool = True) -> list[dict]:
    """Get all self-hosted runners with pagination. Returns empty if no permission."""
    try:
        all_runners = []
        page = 1
        while True:
            data = run_gh_command(
                [f"repos/{repo}/actions/runners?per_page=100&page={page}"]
            )
            runners = data.get("runners", [])
            all_runners.extend(runners)
            if len(runners) < 100:
                break
            page += 1
            if page > 10:  # Safety limit
                break
        if online_only:
            all_runners = [r for r in all_runners if r.get("status") == "online"]
        return all_runners
    except Exception as e:
        print(f"Warning: Cannot access runners API (need admin): {e}")
        return []
```
**EN:** This block defines `get_runners`. Get all self-hosted runners with pagination. It accepts 2 parameter(s): repo, online_only. Internally it emits status messages.
**CN:** 该代码块定义了 `get_runners`。其文档字符串说明了该逻辑的预期职责。它接收 2 个参数：repo, online_only。内部会输出状态信息。

### Lines 153-157: Defines the `parse_time` routine / 定义 `parse_time` 例程
```python
def parse_time(time_str: str) -> datetime:
    """Parse ISO timestamp to datetime."""
    if not time_str:
        return None
    return datetime.fromisoformat(time_str.replace("Z", "+00:00"))
```
**EN:** This block defines `parse_time`. Parse ISO timestamp to datetime. It accepts 1 parameter(s): time_str. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `parse_time`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：time_str。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 160-235: Defines the `calculate_concurrency_metrics` routine / 定义 `calculate_concurrency_metrics` 例程
```python
def calculate_concurrency_metrics(
    jobs: list,
    window_start: datetime,
    window_end: datetime,
    num_runners: int,
) -> dict:
    """Sweep-line algorithm: peak/avg concurrent, saturation time, peak queue."""
    if not jobs:
        return {
            "peak_concurrent": 0,
            "avg_concurrent": 0.0,
            "saturation_seconds": 0,
            "saturation_pct": 0.0,
            "peak_queue": 0,
        }
    window_seconds = (window_end - window_start).total_seconds()
    if window_seconds <= 0:
        return {
            "peak_concurrent": 0,
            "avg_concurrent": 0.0,
            "saturation_seconds": 0,
            "saturation_pct": 0.0,
            "peak_queue": 0,
        }
    running_events = []
    for job in jobs:
        start, end = job["start"], job["end"]
        if end < window_start or start > window_end:
            continue
        running_events.append((max(start, window_start), 1))
        running_events.append((min(end, window_end), -1))
    queue_events = []
    for job in jobs:
        created_at = job.get("created_at")
        started_at = job["start"]
        if created_at and created_at < started_at:
            if started_at < window_start or created_at > window_end:
                continue
            queue_events.append((max(created_at, window_start), 1))
            queue_events.append((min(started_at, window_end), -1))
    running_events.sort(key=lambda e: (e[0], e[1] == 1))
    current_running = 0
    peak_running = 0
    prev_time = window_start
    total_running_seconds = 0.0
    saturation_seconds = 0.0
    for event_time, delta in running_events:
        td = (event_time - prev_time).total_seconds()
        if td > 0:
            total_running_seconds += current_running * td
            if current_running >= num_runners:
                saturation_seconds += td
        current_running += delta
        peak_running = max(peak_running, current_running)
        prev_time = event_time
    if prev_time < window_end:
        td = (window_end - prev_time).total_seconds()
        total_running_seconds += current_running * td
        if current_running >= num_runners:
            saturation_seconds += td
    queue_events.sort(key=lambda e: (e[0], e[1] == 1))
    current_queued = 0
    peak_queue = 0
    for _, delta in queue_events:
        current_queued += delta
        peak_queue = max(peak_queue, current_queued)
    avg_concurrent = total_running_seconds / window_seconds if window_seconds > 0 else 0
    return {
        "peak_concurrent": peak_running,
        "avg_concurrent": avg_concurrent,
        "saturation_seconds": saturation_seconds,
        "saturation_pct": (
            (saturation_seconds / window_seconds * 100) if window_seconds > 0 else 0
        ),
        "peak_queue": peak_queue,
    }
```
**EN:** This block defines `calculate_concurrency_metrics`. Sweep-line algorithm: peak/avg concurrent, saturation time, peak queue. It accepts 4 parameter(s): jobs, window_start, window_end, num_runners. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `calculate_concurrency_metrics`。其文档字符串说明了该逻辑的预期职责。它接收 4 个参数：jobs, window_start, window_end, num_runners。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 238-253: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
_NON_GPU_WORKFLOW_HINTS = (
    "lint",
    "deploy",
    "release",
    "publish",
    "docs",
    "doc",
    "mintlify",
    "runner utilization",  # this very script
    "tag-and-rerun",
    "auto",  # auto-merge etc.
    "label",
    "stale",
    "dependabot",
    "codeql",
)
```
**EN:** This section defines shared constants, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 共享常量，用于把可复用的默认值集中在模块顶部。

### Lines 256-267: Defines the `_likely_no_gpu_jobs` routine / 定义 `_likely_no_gpu_jobs` 例程
```python
def _likely_no_gpu_jobs(workflow_name: str) -> bool:
    """Heuristic: skip per-run job-fetch for workflows that don't dispatch
    to self-hosted GPU runners. The GH API rate limit (~5000 req/hr per
    token) is the bottleneck on busy 24h windows where ~4000 workflow
    runs fire — but only a fraction of those (pr-test, nightly-test,
    pr-test-*kernel, etc.) actually run on GPU runners. Skipping the
    docs/lint/release runs cuts the API call budget by 2-4x.
    """
    if not workflow_name:
        return False
    n = workflow_name.lower()
    return any(h in n for h in _NON_GPU_WORKFLOW_HINTS)
```
**EN:** This block defines `_likely_no_gpu_jobs`. Heuristic: skip per-run job-fetch for workflows that don't dispatch to self-hosted GPU runners. It accepts 1 parameter(s): workflow_name. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `_likely_no_gpu_jobs`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：workflow_name。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 270-490: Defines the `calculate_utilization` routine / 定义 `calculate_utilization` 例程
```python
def calculate_utilization(repo: str, hours: int = 24, runner_filter: str = None):
    """Calculate runner utilization metrics."""

    print(f"Fetching workflow runs from last {hours} hours...")
    all_runs = get_workflow_runs(repo, hours)
    runs = [r for r in all_runs if not _likely_no_gpu_jobs(r.get("name", ""))]
    skipped = len(all_runs) - len(runs)
    print(
        f"Found {len(all_runs)} workflow runs "
        f"({skipped} skipped as non-GPU: docs/lint/release/etc.)"
    )

    # Try to get online runners from API
    print("Fetching online runners...")
    runners = get_runners(repo, online_only=True)

    # Build label -> set of online runner names from API
    api_label_runners = defaultdict(set)
    if runners:
        for runner in runners:
            for label in runner.get("labels", []):
                label_name = label.get("name", "")
                if label_name not in DEFAULT_LABELS_TO_IGNORE:
                    api_label_runners[label_name].add(runner["name"])
        print(f"Got {len(runners)} online runners from API")
    else:
        print("No runner API access, will use observed runners from job data")

    # Track runners seen in jobs (for labels not in API or when API unavailable)
    job_label_runners = defaultdict(set)
    label_jobs = defaultdict(list)  # label -> list of job_info
    # Per-host accumulation: each physical machine appears once regardless of
    # how many overlapping labels it advertises. This is what we use for the
    # "Per Host Utilization" section (the source-of-truth view).
    host_jobs = defaultdict(list)  # runner_name -> list of job_info
    host_labels = defaultdict(set)  # runner_name -> set of labels it ran jobs under

    # Fetch jobs for all runs in parallel. Cap concurrency lower than the
    # GH API secondary rate-limit threshold to avoid bursts that silently
    # drop runs even with retries.
    total_runs = len(runs)
    print(f"Fetching jobs for {total_runs} runs in parallel...")

    def fetch_jobs_for_run(run):
        """Fetch jobs for a single run.

        Returns (run_id, jobs, error_msg). `error_msg` is None on success.
        We surface failures rather than silently dropping the run so the
        caller can report how many runs' jobs are missing — silently
        dropping previously caused 4-gpu-b200 (and every other label) to
        report wildly different numbers depending on transient API hiccups.
        """
        try:
            return (run["id"], get_jobs_for_run(repo, run["id"]), None)
        except Exception as e:
            return (run["id"], None, str(e)[:200])

    all_jobs = []
    failed_runs = []
    # Concurrency=4 with longer retry budget keeps us well below the GH
    # API secondary rate-limit threshold (~10 req/s). On a 24h window
    # with ~1500 GPU-relevant runs (post-filter), this completes in ~5
    # min and almost never hits the rate limit.
    with ThreadPoolExecutor(max_workers=4) as executor:
        futures = [executor.submit(fetch_jobs_for_run, run) for run in runs]
        completed = 0
        for future in as_completed(futures):
            completed += 1
            if completed % 100 == 0:
                print(
                    f"Fetched jobs for {completed}/{total_runs} runs "
                    f"({len(failed_runs)} failed so far)..."
                )
            run_id, jobs, err = future.result()
            if err:
                failed_runs.append((run_id, err))
            elif jobs:
                all_jobs.extend(jobs)

    print(f"Processing {len(all_jobs)} jobs...")
    if failed_runs:
        print(
            f"WARNING: {len(failed_runs)}/{total_runs} runs failed to fetch "
            f"after retries. Utilization will be undercounted. "
            f"First few errors:"
        )
        for rid, err in failed_runs[:5]:
            print(f"  run {rid}: {err}")
    fetch_failure_pct = len(failed_runs) / total_runs * 100 if total_runs > 0 else 0

    for job in all_jobs:
        runner_name = job.get("runner_name")
        if not runner_name:
            continue

        created_at = parse_time(job.get("created_at"))
        started_at = parse_time(job.get("started_at"))
        completed_at = parse_time(job.get("completed_at"))

        if not started_at or not completed_at:
            continue

        duration = (completed_at - started_at).total_seconds()
        queue_time = (started_at - created_at).total_seconds() if created_at else 0
        job_info = {
            "start": started_at,
            "end": completed_at,
            "created_at": created_at,
            "duration": duration,
            "queue_time": queue_time,
            "job_name": job["name"],
            "runner_name": runner_name,
        }

        # Per-host: every job on this physical machine, regardless of label.
        host_jobs[runner_name].append(job_info)

        # Use job labels directly (available in job data)
        job_labels = job.get("labels", [])
        for label in job_labels:
            # Skip generic labels
            if label in DEFAULT_LABELS_TO_IGNORE | GITHUB_HOSTED_LABELS:
                continue
            job_label_runners[label].add(runner_name)
            label_jobs[label].append(job_info)
            host_labels[runner_name].add(label)

    # Merge API runners and job-observed runners
    # Prefer API count (online runners) when available
    all_labels = set(api_label_runners.keys()) | set(job_label_runners.keys())

    # Filter labels if specified
    if runner_filter:
        all_labels = {lbl for lbl in all_labels if runner_filter in lbl}

    print(f"Tracking {len(all_labels)} runner labels: {sorted(all_labels)}")

    window_seconds = hours * 3600
    window_end = datetime.now(timezone.utc)
    window_start = window_end - timedelta(hours=hours)

    # Per-host window-clamped busy time (each physical machine counted once).
    # This is the source of truth for how loaded each host actually is.
    host_busy_seconds = {}
    for host, jobs in host_jobs.items():
        busy = 0.0
        for j in jobs:
            cs = max(j["start"], window_start)
            ce = min(j["end"], window_end)
            if ce > cs:
                busy += (ce - cs).total_seconds()
        host_busy_seconds[host] = busy

    results = []
    for label in sorted(all_labels):
        # Hosts to attribute to this label = union of currently-online
        # runners advertising the label PLUS hosts that actually ran a
        # job under it during the window. The union catches hosts that
        # went offline mid-window (their busy time is still real
        # capacity consumed) and hosts that came online late.
        hosts = api_label_runners.get(label, set()) | job_label_runners.get(
            label, set()
        )
        num_runners = len(hosts) if hosts else 1

        # Pool busy time: sum of busy time across the hosts that could
        # serve this label, regardless of which sibling label actually
        # dispatched the job. This is the right denominator/numerator for
        # asking "how saturated is the underlying hardware that this
        # label depends on?" — sibling labels (e.g. `4-gpu-b200` and
        # `4-gpu-b200-low-disk`) compete for the same physical machines,
        # so their busy time should not be double-counted into separate
        # capacity buckets.
        active_seconds = sum(host_busy_seconds.get(h, 0.0) for h in hosts)
        capacity_seconds = num_runners * window_seconds
        utilization = (
            (active_seconds / capacity_seconds * 100) if capacity_seconds > 0 else 0
        )

        # Job count + queue stats stay label-specific (only jobs that
        # were dispatched under THIS label).
        jobs = label_jobs.get(label, [])
        queue_times = [j["queue_time"] for j in jobs if j["queue_time"] > 0]
        avg_queue = sum(queue_times) / len(queue_times) if queue_times else 0
        max_queue = max(queue_times) if queue_times else 0

        # Concurrency / saturation / queue-depth metrics. Use observed
        # peak as effective capacity if it's lower than the API count
        # (e.g. for autoscaling pools where most listeners sit idle).
        conc_initial = calculate_concurrency_metrics(
            jobs, window_start, window_end, num_runners
        )
        effective_runners = (
            min(num_runners, conc_initial["peak_concurrent"]) or num_runners
        )
        if effective_runners < num_runners and effective_runners > 0:
            conc = calculate_concurrency_metrics(
                jobs, window_start, window_end, effective_runners
            )
        else:
            conc = conc_initial

        results.append(
            {
                "label": label,
                "num_runners": num_runners,
                "effective_runners": effective_runners,
                "num_jobs": len(jobs),
                "total_active_hours": active_seconds / 3600,
                "utilization_pct": utilization,
                "avg_queue_min": avg_queue / 60,
                "max_queue_min": max_queue / 60,
                "peak_concurrent": conc_initial["peak_concurrent"],
                "avg_concurrent": conc["avg_concurrent"],
                "saturation_hours": conc["saturation_seconds"] / 3600,
                "saturation_pct": conc["saturation_pct"],
                "peak_queue": conc["peak_queue"],
            }
        )

    return results, fetch_failure_pct
```
**EN:** This block defines `calculate_utilization`. Calculate runner utilization metrics. It accepts 3 parameter(s): repo, hours, runner_filter. Internally it emits status messages.
**CN:** 该代码块定义了 `calculate_utilization`。其文档字符串说明了该逻辑的预期职责。它接收 3 个参数：repo, hours, runner_filter。内部会输出状态信息。

### Lines 493-596: Defines the `format_report` routine / 定义 `format_report` 例程
```python
def format_report(
    results: list[dict], hours: int, fetch_failure_pct: float = 0.0
) -> str:
    """One compact summary table — original schema, fixed columns.

    Active (hrs) and Utilization now reflect the actual host pool's
    busy time (sum across all jobs on the hosts that advertise this
    label, regardless of which sibling label dispatched them). This
    makes the column meaningful for shared host pools — e.g.
    `4-gpu-b200` and `4-gpu-b200-low-disk` both consume the same
    physical hosts, so their utilization now reflects real hardware
    saturation instead of being divided across labels.
    """
    lines = [
        "# Runner Utilization Report",
        "",
        f"**Time window:** Last {hours} hours · "
        f"**Generated:** {datetime.now(timezone.utc).strftime('%Y-%m-%d %H:%M UTC')}",
        "",
    ]
    if fetch_failure_pct > 1.0:
        lines.append(
            f"⚠️ **Data completeness warning**: {fetch_failure_pct:.0f}% of "
            f"GPU-relevant workflow runs failed to fetch jobs after retries "
            f"(GH API rate limit). Active hours and utilization below are "
            f"under-counted by approximately this fraction."
        )
        lines.append("")
    lines.extend(
        [
            "| Label | Runners | Jobs | Active (hrs) | Utilization | Avg Queue | Max Queue |",
            "|-------|---------|------|--------------|-------------|-----------|-----------|",
        ]
    )
    for r in results:
        bar = "█" * int(r["utilization_pct"] / 10) + "░" * (
            10 - int(r["utilization_pct"] / 10)
        )
        lines.append(
            f"| {r['label']} | {r['num_runners']} | {r['num_jobs']} | "
            f"{r['total_active_hours']:.1f} | "
            f"{r['utilization_pct']:.1f}% {bar} | "
            f"{r['avg_queue_min']:.1f}m | {r['max_queue_min']:.1f}m |"
        )

    # Concurrency Analysis section
    lines.extend(
        [
            "",
            "## Concurrency Analysis",
            "",
            "| Label | Runners (API/Effective) | Peak Concurrent | Avg Concurrent | Saturation Time | Peak Queue |",
            "|-------|-------------------------|-----------------|----------------|-----------------|------------|",
        ]
    )
    for r in results:
        effective = r["effective_runners"]
        avg_pct = (r["avg_concurrent"] / effective * 100) if effective > 0 else 0
        runner_str = (
            f"{r['num_runners']}/{effective}"
            if effective != r["num_runners"]
            else str(r["num_runners"])
        )
        lines.append(
            f"| {r['label']} | {runner_str} | "
            f"{r['peak_concurrent']} | "
            f"{r['avg_concurrent']:.1f} ({avg_pct:.0f}%) | "
            f"{r['saturation_hours']:.1f}h ({r['saturation_pct']:.0f}%) | "
            f"{r['peak_queue']} jobs |"
        )

    # Recommendations
    lines.extend(["", "## Recommendations", ""])
    has_recs = False
    for r in results:
        label = r["label"]
        sat_pct = r["saturation_pct"]
        peak_q = r["peak_queue"]
        effective = r["effective_runners"]
        avg_pct = (r["avg_concurrent"] / effective * 100) if effective > 0 else 0
        if sat_pct > 50 or peak_q > 5:
            lines.append(
                f"⚠️ **{label}**: High saturation ({sat_pct:.0f}%) "
                f"with queue buildup ({peak_q} jobs). Consider adding runners."
            )
            has_recs = True
        elif sat_pct > 20 or peak_q > 0:
            lines.append(
                f"📊 **{label}**: Moderate saturation ({sat_pct:.0f}%), "
                f"peak queue {peak_q} jobs. Monitor for trends."
            )
            has_recs = True
        elif avg_pct < 30 and r["num_jobs"] > 0:
            lines.append(
                f"💡 **{label}**: Low average utilization ({avg_pct:.0f}%). "
                f"Runner pool may be oversized."
            )
            has_recs = True
        else:
            lines.append(f"✓ **{label}**: Healthy utilization with minimal queueing.")
    if not has_recs and results:
        lines.append("All runner pools have healthy utilization.")

    return "\n".join(lines)
```
**EN:** This block defines `format_report`. One compact summary table — original schema, fixed columns. It accepts 3 parameter(s): results, hours, fetch_failure_pct. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `format_report`。其文档字符串说明了该逻辑的预期职责。它接收 3 个参数：results, hours, fetch_failure_pct。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 599-625: Defines the `main` routine / 定义 `main` 例程
```python
def main():
    parser = argparse.ArgumentParser(description="Generate runner utilization report")
    parser.add_argument("--repo", default="sgl-project/sglang", help="GitHub repo")
    parser.add_argument("--hours", type=int, default=24, help="Time window in hours")
    parser.add_argument(
        "--filter", type=str, help="Filter runner labels (e.g., '5090', 'h200')"
    )
    parser.add_argument("--output", type=str, help="Output file (default: stdout)")
    args = parser.parse_args()

    results, fetch_failure_pct = calculate_utilization(
        args.repo, args.hours, args.filter
    )
    report = format_report(results, args.hours, fetch_failure_pct)

    if args.output:
        with open(args.output, "w") as f:
            f.write(report)
        print(f"Report written to {args.output}")
    else:
        print(report)

    # Also write to GITHUB_STEP_SUMMARY if available
    summary_file = os.environ.get("GITHUB_STEP_SUMMARY")
    if summary_file:
        with open(summary_file, "a") as f:
            f.write(report)
```
**EN:** This block defines `main`. It takes no explicit parameters at the top level. Internally it reads or writes files, parses CLI arguments, emits status messages.
**CN:** 该代码块定义了 `main`。它在顶层定义中不接收显式参数。内部会读写文件、解析命令行参数、输出状态信息。

### Lines 628-629: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
```python
if __name__ == "__main__":
    main()
```
**EN:** This guard runs only when the file is executed as a script, making it the operational entrypoint for the module. It hands control to the previously defined helpers.
**CN:** 该保护块只会在文件被直接执行时运行，因此它就是模块的实际入口。 它会把控制流交给前面定义的辅助函数。

## Key Concepts / 关键概念
- **Subprocess orchestration** / 子进程编排
- **CLI parsing** / 命令行解析
- **Environment management** / 环境管理
- **Structured data handling** / 结构化数据处理
- **Git state inspection** / Git 状态检查

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `collections`, `concurrent`, `datetime`, `json`, `os`, `random`, `subprocess`, `time`
