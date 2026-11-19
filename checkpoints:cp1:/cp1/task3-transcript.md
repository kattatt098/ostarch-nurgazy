# Task 3 — Diagnose

 1. Transcript

| # | Evidence file (command) | What I expected | What I found | What I concluded |
|---|---|---|---|---|
| 1 | 00 (README) | List of all available evidence files | 22 files with command outputs | I should investigate the service, timer and script step by step. |
| 2 | 05 (`systemctl --failed`) | Check which services failed | Two failed services: `legacy-sync` and `nightly-report` | Two leads. I need to find which one is related to the ticket. |
| 3 | 06 (`systemctl list-timers`) | Check if the report timer runs every two minutes | `nightly-report.timer` has a next run in about two minutes and activates `nightly-report.service` | The timer is running. The problem may be in the service or script. |
| 4 | 07 (`systemctl status nightly-report.service`) | Check why the report service failed | Service failed with `203/EXEC`. It says No such file or directory. | The service cannot execute the script. |
| 5 | 08 (`systemctl cat nightly-report.service nightly-report.timer`) | Check service and timer configuration | `ExecStart=/usr/local/bin/nightly-report.sh`. Timer uses `OnUnitActiveSec=2min`. | The service points to the correct script path. The timer configuration looks correct. |
| 6 | 09 (`journalctl -u nightly-report.service`) | Find when the problem started | Successful runs on November 9 at 14:18. Failures started at 14:20. | Something changed around 14:20 on November 9. |
| 7 | 10 (`systemctl status legacy-sync.service`) | Check the second failed service | It runs `/bin/false` and has been failed for six days. | This is probably an unrelated old service. |
| 8 | 11 (`journalctl -u legacy-sync.service`) | Check its previous errors | The same failure happened on previous boots. | This service is not the cause of the nightly report problem. |
| 9 | 12 (`journalctl -p err -b`) | Check other system errors | Shows errors from `legacy-sync` and `nightly-report`. | The important error is still the execution failure of the report script. |
| 10 | 13 (`ls -l /usr/local/bin`) | Check if the script exists and has permissions | The script exists and has executable permissions (`-rwxr-xr-x`). | It is not simply a missing file or normal permission problem. |
| 11 | 14 (`ls -l /var/log`) | Check if the report log exists | `nightly-report.log` exists. Its last modification was November 9 at 14:18. | The log has not been updated since the last successful run. |
| 12 | 15 (`tail -n 20 /var/log/nightly-report.log`) | Check the latest report entries | The last entries are from 14:18 on November 9. | The script stopped writing to the log after that time. |
| 13 | 16 (`cat /usr/local/bin/nightly-report.sh`) | Read the script source code | The script contains `date`, `df -h /` and `free -h`. | The script commands look correct. |
| 14 | 17 (`cat -A /usr/local/bin/nightly-report.sh`) | Check hidden characters and line endings | Every line ends with `^M$`. | The file probably uses Windows CRLF line endings. |
| 15 | 18 (`head -1 /usr/local/bin/nightly-report.sh \| od -c`) | Check the first line of the script | The shebang ends with `\r\n` instead of only `\n`. | This confirms that the shebang contains an extra carriage return. |
| 16 | 19 (running the script by hand) | Check if the script works outside systemd | Both normal execution and sudo execution fail with `No such file or directory`. | The problem is inside the script execution, not only in systemd. |
| 17 | 20 (`which bash; ls -l /bin/bash`) | Check if Bash exists | Bash exists at `/bin/bash`. | Bash is installed. The problem is the incorrect shebang ending. |
| 18 | 21 (`tail -n 12 /var/log/apt/history.log`) | Check recent system changes | A full upgrade was performed on November 9 before the problem appeared. | This is a possible timeline clue, but it does not prove the upgrade caused the problem. |
| 19 | 22 (`tail -n 15 ~/.bash_history`) | Check recent administrator actions | The script was copied, chmod was used, and the service was started. | The administrator tried to fix permissions, but permissions were not the real problem. |
| 20 | 01 (`hostnamectl`) | Confirm the server identity | Ubuntu 24.04.1 LTS, hostname `office-srv`. | I am investigating the correct server. |
| 21 | 02 (`uptime`) | Check server uptime and load | Server has been running for almost seven days with very low load. | The server does not appear overloaded. |
| 22 | 03 (`df -h`) | Check available disk space | Root filesystem has 5.8 GB available. | Disk space is not the problem. |
| 23 | 04 (`free -h`) | Check available memory | 1.5 GiB memory is available and swap is unused. | Memory is not the problem. |

 2. Root cause

The root cause is incorrect Windows-style CRLF line endings in `/usr/local/bin/nightly-report.sh`.

Evidence file 17 shows `^M` at the end of every line, and file 18 confirms `\r\n` in the shebang. Linux tries to find `/bin/bash` with an extra carriage return, so it cannot execute the script. This causes error `203/EXEC`.

The timer is working correctly. The problem is not related to disk space, memory or normal file permissions.

 3. Fix

I would fix the script on the server using these commands:

```bash
# 1. Create a backup
sudo cp /usr/local/bin/nightly-report.sh /usr/local/bin/nightly-report.sh.bak

# 2. Convert Windows CRLF to Linux LF
sudo sed -i 's/\r$//' /usr/local/bin/nightly-report.sh

# 3. Set executable permissions
sudo chmod 755 /usr/local/bin/nightly-report.sh

# 4. Check the first line
head -1 /usr/local/bin/nightly-report.sh | od -c

# 5. Clear the previous failed state
sudo systemctl reset-failed nightly-report.service

# 6. Make sure the timer starts automatically after reboot
sudo systemctl enable --now nightly-report.timer

# 7. Start the report service
sudo systemctl start nightly-report.service
```

 4. Verification

I would use the following commands to prove that the fix works.

Step 1: Check the line endings**

```bash
cat -A /usr/local/bin/nightly-report.sh
head -1 /usr/local/bin/nightly-report.sh | od -c
```

Expected result: No `^M` characters. The first line should end with `\n`, not `\r\n`.

Step 2: Check the service**

```bash
sudo systemctl status nightly-report.service
```

Expected result: The latest execution should finish successfully with exit code 0. There should be no `203/EXEC` error.

Step 3: Check the log**

```bash
tail -n 20 /var/log/nightly-report.log
```

Expected result: A new timestamp, filesystem information and memory information should appear in the log.

Step 4: Check the timer

```bash
systemctl is-enabled nightly-report.timer
systemctl is-active nightly-report.timer
systemctl list-timers
```

Expected result:
- Timer is enabled.
- Timer is active.
- The next execution is scheduled approximately two minutes after the previous activation.

**Step 5: Verify after reboot**

First, reboot the server during an approved maintenance window:

```bash
sudo reboot
```

After reconnecting through SSH, wait at least three minutes and run:

```bash
systemctl is-enabled nightly-report.timer
systemctl is-active nightly-report.timer
systemctl status nightly-report.timer
systemctl status nightly-report.service
journalctl -u nightly-report.service --since "5 minutes ago"
tail -n 20 /var/log/nightly-report.log
```

Expected result:

- The timer remains enabled and active after reboot.
- The timer starts automatically.
- The journal shows a successful report execution after reboot.
- The log contains a new timestamp and system information.

If these checks pass, the problem is fixed and the report continues working automatically after reboot.
