# Security & Data Safety Policy

## Supported Versions

Because kernel parameters, block device tunings, and file-system flags directly interact with low-level storage drivers, security vulnerabilities and data-loss risks are prioritized for the latest codebase.

| Version | Supported |
| ------- | --------- |
| Main (latest) | :white_check_mark: |
| Legacy releases | :x: |

## Data Safety & System Vulnerability Reporting

Due to the nature of storage tweaking scripts (modifying `/sys/block/`, dirty ratios, I/O schedulers, or mounting options), improper configuration can lead to filesystem corruption, kernel panics, data loss upon sudden shutdown, or security bypasses on encrypted partitions.

If you identify a bug or tweak in this repository that causes severe system instability, data loss risks, or security vulnerabilities, **please do not report it in a public issue**.

### Reporting Process

1. **Private Disclosure:** Create a private security advisory on GitHub or email the repository maintainer directly.
2. **Provide Details:**
   * **Device Details:** Storage type (eMMC vs UFS version), Kernel version, Android release, and root solution (Magisk/KernelSU/APatch).
   * **Vulnerability / Issue Description:** Details on how the tweak introduces kernel instability, data loss, or partition corruption risks.
   * **Steps to Reproduce:** Exact steps or command scripts that trigger the issue.
   * **Logs:** Relevant `dmesg`, `pstore`, or `logcat` outputs (redact any sensitive user information before submitting).

### Response Expectations

* **Initial Response:** Within 48–72 hours of report submission.
* **Triage & Patching:** High-priority patches affecting data safety or severe system instability will be released as immediate updates.
* **Credit:** Researchers and contributors reporting validated vulnerabilities or safety risks will be acknowledged in release notes (unless anonymity is requested).

Thank you for helping keep low-level storage modifications safe and reliable!
