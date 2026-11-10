# eMMC / UFS Storage Tune

Two Magisk modules that tune Android storage for faster loading of large files (game assets and similar).

| Module | Use on |
|---|---|
| `emmc_tune.zip` | Devices with eMMC storage (`mmcblk0`) |
| `ufs_tune.zip` | Devices with UFS storage (`sda` and others). Does nothing on eMMC. |

Values are applied at boot and again after 1 and 3 minutes (some vendor kernels reset them), then the script exits. Nothing keeps running or polling. Edit `config.txt` to change a value. Leave a value empty, or write `keep`, to skip it.

---

## eMMC settings (`emmc_tune`)

| Setting | Default | Notes |
|---|---|---|
| `DEVICE` | `auto` | First `mmcblk` device |
| `READ_AHEAD_KB` | 2048 | Also applied to `dm-*` and `loop*` devices |
| `NR_REQUESTS` | 512 | Returns diminish above this |
| `MAX_SECTORS_KB` | `max` | Device maximum request size |
| `SCHEDULER` | `none` | Applied only if the kernel has it |
| `WBT_LAT_USEC` | empty (stock) | Write-back throttling. 0 turns it off, which can make reads slower during heavy writes |
| `IOSTATS`, `ADD_RANDOM` | 0 | Small CPU savings per I/O |
| `RQ_AFFINITY` | 2 | Small CPU savings per I/O |
| `CLK_SCALING` | 0 | eMMC at full speed, small battery cost |
| `VFS_CACHE_PRESSURE` | 30 | Keeps file and folder lookup cache longer (stock 100) |
| `DIRTY_EXPIRE_CENTISECS` / `DIRTY_WRITEBACK_CENTISECS` | 3000 / 1500 | Batches writes into fewer, larger ones |
| `FSTRIM` | 0 | Set to 1 to run `fstrim /data` once per boot |

## UFS settings (`ufs_tune`)

Same as above, with these differences:
- `DEVICES=auto` tunes every `sd*` block device that sits on a UFS host.
- `READ_AHEAD_KB=auto` picks 512 below 3GB of RAM, 1024 at 3 to 6GB, and 2048 above that. You can also set a number.
- `NR_REQUESTS` defaults to 256.
- UFS clock scaling (`clkscale_enable`) is skipped by default and clock gating (`clkgate_enable`) is skipped by default. These are the Qualcomm UFS paths, and any missing path is skipped.
- `UFS_GOVERNOR` is empty by default. `performance` is the usual value if you want it.
- UFS write booster is left out on purpose, because it uses an SLC buffer that adds wear and varies between chips.

---

## Check that it works (Termux)

```
su -c 'cat /sys/block/mmcblk0/queue/read_ahead_kb'
su -c 'cat /sys/block/mmcblk0/queue/nr_requests'
su -c 'cat /sys/block/mmcblk0/queue/scheduler'
su -c 'cat /sys/block/mmcblk0/queue/max_sectors_kb'
su -c 'cat /sys/block/mmcblk0/queue/wbt_lat_usec'
su -c 'cat /sys/class/mmc_host/mmc0/clk_scaling/enable'
su -c 'cat /proc/sys/vm/vfs_cache_pressure'
```

On UFS, replace `mmcblk0` with your device (for example `sda`). If a path doesn't exist on your kernel, that setting is skipped and the rest still apply.

---

## Q&A

**What is read-ahead?**
`read_ahead_kb` controls how much extra data the kernel reads past what an app asked for, on the guess that it will be needed next. A bigger value helps with large files read start to finish. The extra data is cached in RAM.

**Does it reduce storage lifespan?**
No. Flash wear comes from writing and erasing, and reads don't wear it. Read-ahead writes nothing.

**Is it kernel level? Does it affect all apps?**
Yes. It's a block-layer setting, so it applies to every file read from that device by any app. Apps that read small scattered pieces waste a little RAM and I/O, though the kernel shrinks the window for random access on its own. It can't be set for one app only. If other apps feel slower or RAM gets tight, lower it.

**Can the Snapdragon 680 handle 1024? More?**
The CPU isn't the limit, because it's a kernel setting with almost no CPU cost. The limits are RAM and storage speed.
- 512 to 1024: safe.
- 2048: works, and gave a noticeable improvement on the test device, but it uses more RAM per read.
- 4096 and up: usually slower, because the kernel reads far more than needed.

**Do I have to wait after reboot before using a game?**
No. The first apply runs as soon as boot finishes. The later re-applies only cover the case where the system resets the value.

**Why don't manufacturers tune it like this?**
They tune for the average user and the worst case.
- Larger read-ahead and a lower `vfs_cache_pressure` hold more data in RAM, which can mean more app kills on a 4GB device.
- The gain is biggest for large sequential reads, while many apps read small scattered chunks.
- Reading data that is never used costs a little power.
- Budget devices get little storage tuning, and retesting thousands of apps is expensive.

On your own device you know the workload. Over a few days, watch for apps reloading after you switch away (RAM too tight) and new stutter. If you see them, lower read-ahead from 2048 to 1024 first.

**Why does `nr_requests` read 31 instead of my value?**
With the `none` scheduler, `nr_requests` is capped by the hardware queue depth (31 on many eMMC chips), so a larger value is rejected and skipped. To use a deeper queue (for example 512), set `SCHEDULER=mq-deadline` (or another scheduler your kernel lists). Test both and keep what loads faster.

**Why do some paths say "Invalid argument" or "No such file"?**
`wbt_lat_usec` only works when write-back throttling is active for the current scheduler, and `clk_scaling` only exists on kernels that expose it. A missing or unreadable path is skipped, and the other settings still apply.

**Can I tune it more?**
Only a little. 2048 read-ahead and 512 `nr_requests` are near the ceiling. Going lower than 30 on `vfs_cache_pressure` gains very little, and below 10 the kernel hoards cache it doesn't need.

Left out on purpose:
- `nomerges` (the default is right).
- Anything that disables barriers or the write cache, because those risk data loss.

**Is it universal across manufacturers?**
The ideas are. The exact settings and paths are not.
- Works almost everywhere: `read_ahead_kb`, `nr_requests`, `max_sectors_kb`, `iostats`, `add_random`, `rq_affinity`, `vfs_cache_pressure`, `dirty_*`.
- Varies: the block device name, the clock scaling path, `wbt_lat_usec` support, the available schedulers, and F2FS vs ext4.
- Vendor kernels sometimes reset values at boot, which is why the module re-applies.
- The numbers are tuned for a 4GB device. On a 2GB device, 2048 read-ahead could cause memory pressure.

**Is `emmc_tune` universal for eMMC?**
Mostly. `DEVICE=auto` picks the first `mmcblk` device, and a setting whose path doesn't exist is skipped. `clk_scaling` uses the Qualcomm-style path `/sys/class/mmc_host/mmc0/clk_scaling`, so MediaTek or Exynos devices may not expose it.

**Does it affect storage health?**
It's safe, and slightly helpful. Reads don't wear flash, and batching writes reduces wear a little. The costs are heat and battery (clock scaling off, write throttling off), plus up to about 30 seconds of unsaved writes lost on a crash or power loss.

**Does it affect the external SD card?**
Barely. `DEVICE=auto` picks `mmcblk0` (the internal eMMC), and the SD card is usually `mmcblk1`. `clk_scaling` only targets `mmc0`. The dirty-writeback settings are global, so they also apply to writes to the SD card. Always eject the card before removing it.
