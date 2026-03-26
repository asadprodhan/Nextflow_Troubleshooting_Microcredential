# Nextflow Troubleshooting Microcredential (Single Workstation)

**Asad Prodhan**  
The University of Western Australia (UWA)  
Perth, Western Australia, Australia  

---

## Learning Objective

Diagnose and resolve stalled Nextflow pipelines caused by CPU, memory, and disk I/O contention in a single-workstation environment.

---

## Step 1 — Check pipeline status

```bash
ps -ef | grep -i nextflow | grep -v grep
```

**Interpretation**
- Processes present → pipeline running  
- No process → pipeline stopped  

---

## Step 2 — Diagnose system resources

```bash
uptime
nproc
free -h
vmstat 2 5
```

### `uptime`
- Shows system load average (1, 5, 15 min)
- Compare with `nproc` (CPU cores)

**Interpretation**
- Load ≈ cores → fully utilised  
- Load >> cores → overloaded  

---

### `free -h`
Example:
```
              total   used   free   buff/cache   available
Mem:          125Gi   38Gi   19Gi   68Gi         87Gi
Swap:         8.0Gi   8.0Gi  0Gi
```

**Key fields**
- `used` → actively used memory  
- `free` → unused memory  
- `buff/cache` → filesystem cache (GOOD, not a problem)  
- `available` → real usable memory  
- `Swap` → overflow memory  

**Interpretation**
- High swap usage → memory pressure ⚠️  
- High cache → normal (do not clear)  
- Low available → risk of swapping  

---

### `vmstat 2 5`
Example columns:
```
r  b   swpd   free   si   so   wa
6 15 8383860 20786332 722 1065 48
```

**Key fields**
- `r` → runnable processes (CPU demand)  
- `b` → blocked processes (I/O wait)  
- `swpd` → swap used  
- `si/so` → swap in/out  
- `wa` → I/O wait  

**Interpretation**
- `wa > 20–30%` → disk bottleneck ⚠️  
- High `b` → processes waiting on disk  
- High `si/so` → active swapping (bad)  
- High `r` → CPU contention  

---

## System Bottleneck Flow

```
        [High CPU Load]
               ↓
        [RAM Saturation]
               ↓
        [Swap Usage ↑]
               ↓
        [Disk I/O Wait ↑]
               ↓
        [Processes Blocked]
               ↓
        [Pipeline Appears Stalled]
```

**Key Insight:** This represents a system-level bottleneck, not a pipeline error.

---

## Step 3 — Identify heavy pipelines

```bash
ps -ef | grep nextflow
```

**Interpretation**
- Assembly + BLAST → resource intensive  
- BLAST-only → comparatively lighter  

---

## Step 4 — Reduce contention

```bash
kill -9 <PID>
```

**Interpretation**
- Frees CPU, memory, and disk I/O  
- Improvement confirms resource contention  

---

## Step 5 — Confirm recovery

```bash
free -h
vmstat 2 5
```

**See how to interpret `free` and `vmstat` outputs in Step 2.**

**Healthy signs**
- Swap stabilised  
- Reduced I/O wait (`wa`)  
- CPU doing real work (`us` increases)  

---

## Step 6 — Check active jobs

```bash
pgrep -af '^blastn '
```

**Interpretation**
- Displays active BLAST processes  
- Confirms execution is ongoing  

---

## Step 7 — Monitor progress

```bash
find work -name ".exitcode" | wc -l
```

**Interpretation**
- Increasing count → pipeline progressing  
- Static count → tasks still running or waiting  

---

## Step 8 — Check task results

```bash
find work -name ".exitcode" -exec cat {} \;
```

**Interpretation**
- `0` → successful execution  
- `130 / 143` → interrupted tasks (not intrinsic errors)  

---

## Step 9 — Check logs

```bash
tail -f .nextflow.log
```

**Interpretation**
- `Submitted` → tasks launching  
- `ERROR` → actual failure  
- `SIGINT` → manual interruption  

---

## Step 10 — Resume safely

```bash
nextflow run main.nf ... -resume
```

**Interpretation**
- Reuses completed tasks  
- Restarts only incomplete or interrupted tasks  

---

## Key Learning Points

- Slow execution does not necessarily indicate failure  
- Swap usage and disk I/O are major bottlenecks  
- Avoid running multiple resource-intensive pipelines simultaneously  
- Always use `-resume` to recover interrupted workflows  
- A single workstation requires manual resource management  

---

## Recommended Settings

| Scenario | Settings |
|--------|---------|
| BLAST only | `--cpus 6 --max_forks 2` |
| Assembly pipeline | `maxForks = 1` |
| Mixed pipelines | Not recommended |

---

## Teaching Insight

This exercise demonstrates:
- Systems-level thinking in bioinformatics workflows  
- Importance of resource-aware pipeline design  
- Practical debugging beyond code-level errors  
