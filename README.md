# rust-ci-stability


# 🦀 Rust CI Stability: OOM & Disk Pressure Mitigation

## Overview

This repository demonstrates how to diagnose and stabilize intermittent Rust `cargo build` and `cargo test` failures caused by **out-of-memory (OOM) conditions** and **disk pressure from stale build artifacts**.

The failures manifested as:

* `Killed` processes
* LLVM crashes during compilation
* Flaky test failures in CI environments with limited resources

The solution focuses on **system-level mitigations** combined with **Rust-specific build controls** to ensure reliable CI execution.

---

## Problem Summary

Intermittent CI failures were observed in a Rust workspace due to:

* **Parallel compilation exhausting available RAM**
* **Nearly full disk space caused by accumulated build artifacts**
* Limited resources on CI runners exacerbating both issues

These conditions led to nondeterministic failures that were difficult to reproduce locally.

---

## Solution Strategy

The CI pipeline was stabilized using a layered mitigation approach:

### 1️⃣ Free Disk Space

* Remove stale Rust build artifacts (`target/`)
* Clean Cargo build outputs
* Optionally remove large cached Cargo registry data
* Log disk usage before and after cleanup to verify improvement

### 2️⃣ Add Swap Space

* Create and enable a swapfile on the CI runner
* Increase available virtual memory to prevent OOM kills
* Validate swap activation using system diagnostics

### 3️⃣ Limit Build Parallelism

* Restrict Rust build jobs to reduce peak memory usage
* Limit test thread concurrency to avoid runtime memory spikes
* Apply limits both in CI and persistently via Cargo configuration

### 4️⃣ Validate Stability

* Rebuild the project after mitigations
* Run tests multiple times to confirm reliability
* Collect post-run diagnostics to ensure no OOM or disk saturation events occurred

---

## CI Implementation

The GitHub Actions workflow performs the following steps:

1. **Pre-flight diagnostics**

   * CPU, memory, disk, and swap inspection
2. **Disk cleanup**

   * Remove stale build artifacts and caches
3. **Swapfile creation**

   * Enable swap to mitigate memory pressure
4. **Controlled build**

   * Limit Cargo build jobs
5. **Repeated test execution**

   * Run `cargo test` multiple times to verify stability
6. **Post-run diagnostics**

   * Check for OOM events and disk usage regressions

---

## Persistent Configuration

To ensure stability beyond CI, Cargo build limits are also defined in:

```
.cargo/config.toml
```

```toml
[build]
jobs = 2

[env]
RUST_TEST_THREADS = "1"
```

This ensures consistent behavior across local and CI environments.



## Results

After applying these mitigations:

* CI builds complete reliably without OOM failures
* Tests pass consistently across repeated runs
* Disk usage remains below critical thresholds
* No LLVM crashes or killed processes observed



## Notes

* Swapfiles on GitHub-hosted runners are ephemeral and created per job
* For long-running or heavier workloads, **persistent swap on self-hosted runners is recommended**
* This approach generalizes well to other resource-constrained Rust CI environments



## Key Takeaways

* Rust’s parallel compilation can overwhelm small CI runners
* Disk pressure and memory pressure often combine to cause flakiness
* System-level mitigations + build-level controls are essential for stability
* Diagnostics and repeat testing are critical for validating fixes




