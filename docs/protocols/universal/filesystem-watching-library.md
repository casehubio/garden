---
id: PP-20260616-f5a372
title: "Use io.methvin:directory-watcher for filesystem watching — not raw WatchService"
type: rule
scope: universal
applies_to: "Any Java project that needs to detect filesystem changes (file create, modify, delete) in a directory tree"
severity: guidance
refs: []
violation_hint: "Direct use of java.nio.file.WatchService or org.apache.commons.io.monitor.FileAlterationMonitor for recursive directory watching"
created: 2026-06-16
---

Use `io.methvin:directory-watcher` for recursive filesystem change detection.

```xml
<dependency>
    <groupId>io.methvin</groupId>
    <artifactId>directory-watcher</artifactId>
    <version>0.19.1</version>
</dependency>
```

**Why not raw `WatchService`:**
- macOS JDK `WatchService` uses polling (JDK-7133447) — high latency, high CPU
- No recursive watching on any platform — requires manual subdirectory registration and race-prone bookkeeping
- No duplicate event deduplication — Windows and macOS emit redundant modify events
- No file hashing — cannot distinguish a real content change from a metadata touch

**What directory-watcher provides:**
- Native macOS FSEvents via JNA (`MacOSXListeningWatchService`) — low latency, no polling
- Recursive watching on all platforms (native on macOS/Windows, auto-registration on Linux)
- Built-in file content hashing to deduplicate spurious events (configurable, on by default)
- Simple listener API: `DirectoryWatcher.builder().path(dir).listener(event -> ...).build()`
- `watchAsync()` for non-blocking background watching

**Also avoid:** Apache Commons IO `FileAlterationMonitor` — polling-only, no native integration, heavier dependency tree.

**GraalVM native image note:** The JNA-based macOS implementation may need reflection configuration for native builds. On Linux (typical production), directory-watcher delegates to the standard JDK `WatchService` (inotify-based), which works in native image without configuration.
