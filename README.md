# Proxmox Disk Health & I/O Pressure Diagnostic Guide

Author: Abubakkar Khan |
Designation: System Admin |
Date: July 30, 2025

## Overview

This guide is intended to help system administrators diagnose and monitor disk health and I/O pressure in Proxmox environments — including both the host node and running LXC containers. We cover the essential tools, their purpose, and how to interpret results to assess potential I/O bottlenecks or disk issues.

---

## 🔧 Tools & Commands

### 1. `iostat` – Disk I/O Statistics

**Command:**

```bash
apt install sysstat -y   # Install if not present
iostat -xz 1 5
```

**What to look for:**

* `%util`: Near 100% indicates the device is saturated.
* `await`: High values (over 20–30ms) can signal latency.
* `r_await`, `w_await`: Separates read/write wait times.
* `aqu-sz`: Average queue size — high means requests are piling up.

📌 Use this to pinpoint slow disks or those with high contention.

---

### 2. `cat /proc/pressure/io` – I/O Pressure Stall Information

**Command:**

```bash
cat /proc/pressure/io
```

**What to look for:**

* `some`: Time when at least one task stalled due to IO.
* `full`: Time when all non-idle tasks stalled.
* `avg10`, `avg60`, `avg300`: Percent of time in stall over the last 10, 60, and 300 seconds.

📌 `avg10` > 10% means considerable pressure. Over 30% = critical.

---

### 3. `smartctl` – Disk Health SMART Data

**Command:**

```bash
apt install smartmontools -y
smartctl -a /dev/sdX     # Replace X with your disk letter
```

**What to look for:**

* `Reallocated_Sector_Ct`, `Pending_Sector`, `Uncorrectable_Error`: Non-zero is bad.
* `Temperature`: Keep below 50°C under load.
* `Power_On_Hours`, `Wear_Leveling_Count`: Good for aging analysis.

📌 Failing values or pre-fail attributes often show impending disk failure.

---

### 4. `df -h` – Disk Usage

**Command:**

```bash
df -h
```

**What to look for:**

* Any partition above 85% is a red flag.
* Proxmox root (`/`) or container volumes nearing full can cause hangs or failures.

📌 Plan cleanup or expansion if usage is high.

---

### 5. `du -sh /path/*` – Identify Large Directories

**Command:**

```bash
du -sh /var/lib/* | sort -h
```

**What to look for:**

* Find which directories are consuming the most space.

📌 Useful when `df` shows high usage, but source is unclear.

---

### 6. `zfs list` (if using ZFS)

**Command:**

```bash
zfs list
```

**What to look for:**

* `USED` and `AVAIL`: Monitor dataset growth.

📌 ZFS can delay writes under pressure — track proactively.

---

### 7. For LXC Containers – Check from Host

**Command:**

```bash
pct exec <CTID> -- iostat -xz 1 5
pct exec <CTID> -- df -h
```

**What to look for:**

* Same metrics as above but scoped to container disks.

📌 Identifies container-level I/O bottlenecks.

---

### 8. `dmesg` – Kernel Messages (Hardware/IO Errors)

**Command:**

```bash
dmesg | grep -iE 'error|fail|warn|ata|nvme|io|ext4'
```

**What to look for:**

* Disk timeouts, bus errors, file system warnings.

📌 Review regularly — early signs of disk failure often show here first.

---

## 🔀 Routine Checklist

| Task                      | Frequency |
| ------------------------- | --------- |
| Run `iostat`              | Daily     |
| Check `/proc/pressure/io` | Hourly    |
| Run `smartctl`            | Weekly    |
| Monitor disk space        | Daily     |
| Review `dmesg`            | Weekly    |

---

## ⚠️ Alerts & When to Act

| Indicator                   | Threshold | Action                         |
| --------------------------- | --------- | ------------------------------ |
| Disk %util                  | > 90%     | Investigate I/O-heavy services |
| SMART reallocated sectors   | > 0       | Plan for disk replacement      |
| Pressure avg10 (`full`)     | > 30      | Major I/O starvation           |
| `/` disk usage              | > 85%     | Clean logs/backups             |
| `dmesg` hardware I/O errors | Any       | Backup and replace hardware    |

---

## 🧠 Tips

* Always monitor host first, then container I/O.
* Avoid co-locating heavy I/O workloads.
* Use SSDs or NVMe for write-heavy apps.
* Consider ZFS ARC tuning if using ZFS.

---

## ✅ Summary

This guide helps you proactively diagnose, monitor, and troubleshoot disk-related performance issues and health in Proxmox. Regular monitoring is essential to prevent critical outages.

Stay ahead. Monitor smartly.

---

© 2025 Abubakkar | System Admin
