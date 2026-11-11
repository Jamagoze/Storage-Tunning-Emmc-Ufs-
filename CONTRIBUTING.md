# Contributing to Storage Tuning (eMMC / UFS)

Thank you for your interest in contributing to **Storage-Tunning-Emmc-Ufs-**! This project aims to provide safe, effective, and benchmarked storage performance tweaks (I/O schedulers, read-ahead buffers, file-system params, and kernel sysfs tunings) for eMMC and UFS storage chips on Android/Linux systems.

Because low-level storage modifications directly impact system stability, data integrity, and battery efficiency, we ask that all contributors follow these guidelines.

---

## Code of Conduct

Please maintain a constructive, respectful, and inclusive environment for everyone. Treat all contributors and issue submitters with courtesy.

---

## What Can You Contribute?

- **Kernel & Sysfs Tweaks:** Propose optimized values for parameters like `read_ahead_kb`, I/O schedulers (`BFQ`, `Kyber`, `mq-deadline`, `none`), dirty memory ratios, or block device settings.
- **Hardware Profile Additions:** Add specific configurations optimized for different eMMC standards (e.g., eMMC 5.1) or UFS generations (UFS 2.1, 3.1, 4.0).
- **Bug Reports & Data Integrity Risks:** Report issues where tweaks cause device lag, high battery drain, data corruption risks, or system freezes.
- **Benchmarking & Validation:** Provide reproducible benchmark results (e.g., using `fio`, CPDT, or AndroBench) verifying the efficacy of proposed changes.

---

## Reporting Issues & Tweaks

When reporting issues or proposing custom tweaks, please include:

1. **Storage Type & Hardware:** (e.g., eMMC 5.1, UFS 2.1, UFS 3.1, UFS 4.0)
2. **Device & Chipset / Kernel:** (e.g., Xiaomi Poco F3, Snapdragon 870, Kernel 4.19 / 5.4 / 6.1)
3. **Execution Environment:** (e.g., Magisk module, KernelSU, init.d script, customized kernel build)
4. **Relevant System Info:** Provide output of block dev parameters if applicable:
   ```bash
   cat /sys/block/sd*/queue/scheduler
   cat /sys/block/sd*/queue/read_ahead_kb
   ```

---

## Pull Request Guidelines

1. **Fork & Branch:** Create a dedicated branch off `main` for your changes:
   ```bash
   git checkout -b feature/ufs-3.1-optimization
   ```
2. **Provide Benchmarks:** Pull requests that modify low-level sysfs values or I/O schedulers **must** include before-and-after performance/power metrics (sequential read/write, random 4K I/O, latency impact).
3. **Focus on Data Safety:** Never propose default settings that disable synchronous flushes or pose severe data corruption risks on sudden power loss (e.g., extreme `dirty_expire_centisecs` tuning without caveats).
4. **Clean Commits:** Keep your commits descriptive and focused on single logical changes (e.g., `tune: adjust read_ahead_kb for UFS 3.0 devices`).

---

## Development & Scripting Rules

- **POSIX Compatibility:** Ensure shell scripts (`.sh`) are POSIX-compliant or explicitly compatible with Android's `mksh` / `toybox` / `busybox` environment.
- **Graceful Fallbacks:** Scripts must check for path existence (e.g., `[ -f /sys/block/... ]`) before attempting to write values, avoiding console errors across different Android kernel revisions.

Thank you for helping optimize storage performance safely!
