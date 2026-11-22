# Lab 6 — Live processes and memory pressure

## What I did

### 1. One process in three tools: ping

Windows (PowerShell):
```
Start-Process ping -ArgumentList '-t','127.0.0.1'
Get-CimInstance Win32_Process -Filter "Name='PING.EXE'" | Select-Object ProcessId, ParentProcessId, WorkingSetSize
(Get-Process ping).HandleCount
```
Linux:
```
ping 127.0.0.1 > /dev/null &
ps -o pid,ppid,rss,cmd -C ping
sudo ls /proc/$(pgrep -x ping)/fd | wc -l
```
Then htop on Linux, Task Manager and Process Explorer on Windows.

### 2. Memory pressure on Linux
A python3 process allocated memory until the kernel killed it. The exact command was not
saved in the shell history, so it is not shown here (the process is PID 1587 in
kernel-log-extract.txt). Memory was sampled every 3 seconds (`free` and `vmstat`) before,
during and after; the output is in memory-pressure.txt. The kernel log was then searched:
```
sudo journalctl -k --since "09:10" | grep -iE "out of memory|oom|killed process"
```

### 3. Memory pressure on Windows
A PowerShell loop allocated 100 MB `byte[]` chunks (up to 2500 MB), checking
`Win32_OperatingSystem.FreeVirtualMemory` each time, then held the memory for 90 s.

## Result

### Three tools, one process (ping)

| | Windows | Linux |
|---|---|---|
| PID | 10304 | 23585 |
| Parent PID | 9960 (the PowerShell that launched it) | 1554 |
| Memory | Working Set 7 749 632 bytes (about 7.4 MB) | RSS 2404 KB |
| Handles / file descriptors | 82 handles | 5 file descriptors |
| Tools | Task Manager, Process Explorer | htop |

htop also shows a second ping (PID 23602), an extra one I started by mistake. The measurements are for PID 23585.

Where the tools disagree or use different names:
- Memory: Windows reports Working Set (bytes in WMI, KB in Task Manager), Linux reports RSS/RES. Similar idea, different names and units.
- Parent PID: Task Manager has no parent column. It is visible in Process Explorer (tree) and in WMI. htop shows it in tree mode (F5).
- Handles vs descriptors: 82 Windows handles cover many object types (registry keys, events, mutexes); 5 Linux descriptors are only files and sockets, so the numbers are not comparable.

### Linux memory pressure (memory-pressure.txt)

| Moment | RAM used | RAM free | Swap |
|---|---|---|---|
| before (09:21:09) | 256 Mi | 1.4 Gi | 0 B used |
| during, 09:23:04 | 1133 MB | 600 MB | 0 |
| during, 09:23:10 | 1699 MB | 33 MB | swapping starts |
| during, 09:23:34 | 1904 MB | 56 MB | about 2.0 GB in swap (vmstat) |
| after (09:23:53) | 219 Mi | 1.7 Gi | 9.4 Mi used |

RAM filled up in about 15 seconds, then the machine started swapping and the free memory stayed near 30-50 MB.
At 09:23:37 the kernel killed the process and all the memory came back.

Kernel log (kernel-log-extract.txt), 09:23:38:
`python3 invoked oom-killer` → `Out of memory: Killed process 1587 (python3)`, anon-rss 1769728 kB.

### Windows memory pressure

| | Before | After |
|---|---|---|
| Page file C:\pagefile.sys | 728 MB | 3756 MB |
| Virtual memory limit | 4 931 208 KB | 8 031 292 KB |

The script allocated all 2500 MB. Minimum free virtual memory during the run was 257 MB.
Free memory jumped up four times (352→740, 335→788, 270→771, 257→815 MB): Windows grew the page file
automatically. Peak page file usage was 386 MB. The limit was never reached.

Windows event log: no `Microsoft-Windows-Resource-Exhaustion-Detector` events were found
(`NoMatchingEventsFound`). This is the Windows counterpart of the Linux OOM record. It was not written
because Windows never ran out of memory: the page file grew instead of the allocation failing.

## What did not work the first time

- Linux: `sudo` rejected my password twice, so one log extract came out empty. `journalctl -k` worked without sudo.
- Linux: `ls /proc/PID/fd` for ping gave "Permission denied" (ping runs with elevated privileges), and
  a leftover earlier ping gave a wrong count (55). Fixed by killing the old ping and using sudo: 5 descriptors.
- Windows: I expected the system to run out of memory and log an event. It did not, because the page file
  is system-managed and grew from 728 MB to 3756 MB. To force the event, the page file would have to be fixed
  at 728 MB first (not done).

## Evidence

- [evidence/task-manager.png](evidence/task-manager.png) — ping in Task Manager (Details tab)
- [evidence/process-explorer.png](evidence/process-explorer.png) — the same ping in Process Explorer
- [evidence/htop.png](evidence/htop.png) — ping in htop on Linux
- [evidence/ping-windows.txt](evidence/ping-windows.txt) — PID, parent, working set, handles on Windows
- [evidence/ping-linux.txt](evidence/ping-linux.txt) — PID, parent, RSS, descriptors on Linux
- [evidence/memory-pressure.txt](evidence/memory-pressure.txt) — memory before, during, after on Linux
- [evidence/kernel-log-extract.txt](evidence/kernel-log-extract.txt) — oom-killer records from the kernel log
- [evidence/windows-load-run1.txt](evidence/windows-load-run1.txt) — output of the Windows allocation loop
- [evidence/pagefile-after-run1.txt](evidence/pagefile-after-run1.txt) — page file size and virtual memory after the Windows run
