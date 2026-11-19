# Task 2 — Plan and justify

Allocation

| | vCPUs | Memory | Disk | Network adapter and mode |
|---|---|---|---|---|
| Ubuntu Server | 1 | 1 GiB | 15 GiB, dynamically allocated | one adapter, Shared network (NAT with host access) |
| Windows 11 | 2 | 4 GiB | 64 GiB, dynamically allocated | one adapter, Shared network |

Arithmetic

Host figures from Task 1: Apple M1, 8 cores, 8 GiB RAM (8,192 MiB), 68.0 GB free on the system disk (63 GiB available per `df`).

| Resource | Guests | Host total | Left for the host |
|---|---|---|---|
| CPU | 1 + 2 = 3 vCPU | 8 cores | 5 cores |
| Memory | 1,024 + 4,096 = 5,120 MiB | 8,192 MiB | 3,072 MiB (3 GiB) |
| Disk, nominal | 15 + 64 = 79 GiB | 63 GiB available | does not fit if both disks fill up |
| Disk, actual now | about 25 GB (Ubuntu 5.62 GB, Windows 19.35 GB) | 68 GB free | about 43 GB |

Assumption for the host: macOS with the hypervisor needs at least 3 GiB of memory and 20 GB of free disk. Measured with ordinary applications open: 7,566 MB used, of which 3,425 MB compressed, so the host is already under memory pressure and every extra GiB given to a guest costs responsiveness.

This computer can run both guests at the same time, but only at these minimum sizes. The 2 GiB + 4 GiB used in the labs would leave the host 2 GiB, which is not usable.

 Justification

- Ubuntu Server 1 vCPU, 1 GiB, 15 GiB: the server edition has no desktop and needs about 1 GiB; going to 2 GiB (as in the labs) would leave the host only 2 GiB. One vCPU is enough for ssh and light work.
- Windows 11 2 vCPU, 4 GiB, 64 GiB: these are Microsoft's minimum requirements; going lower is not supported and would make the guest swap heavily, so I did not set anything below a minimum.
- Dynamically allocated disks: the nominal 79 GiB does not fit in 63 GiB of free space, so fixed-size disks cannot be created at all. Dynamic disks only take what is written (about 25 GB now). The risk is that the guests grow until the host disk is full; I accept it and watch free space with `df -h /` on the host.
- Network, Shared (NAT with host access) for both: the guest reaches the internet through the host, and the host can reach the guest by IP, which is what ssh needs, with one adapter. Alternatives: Bridged gives the guest its own MAC address and the teaching network filters by MAC; Host-only has no internet; Emulated VLAN with port forwarding also works but needs a rule for every port.
- vCPU split 1 + 2 of 8: leaves 5 cores for the host, which has no CPU problem; memory is the real limit.

Verification plan

| Where | Command | Expected output |
|---|---|---|
| Ubuntu | `nproc` | `1` |
| Ubuntu | `free -m` | total about 900-1000 MB |
| Ubuntu | `lsblk` | one 15 GiB disk, mounted at `/` |
| Ubuntu | `ip -4 a` | an address 192.168.x.x/24 on the network interface |
| Ubuntu | `ping -c 3 8.8.8.8` | 0% packet loss |
| Host (Mac) | `ssh user@192.168.x.x` | login prompt of Ubuntu, then a shell |
| Windows | `Get-CimInstance Win32_ComputerSystem \| Select NumberOfLogicalProcessors, TotalPhysicalMemory` | 2 and about 4,294,967,296 (4 GiB, or slightly less) |
| Windows | `Get-Disk` | one 64 GiB disk |
| Windows | `Test-Connection 8.8.8.8` | replies with a response time |
| Host (Mac) | `top -l 1 \| grep PhysMem` | the host still shows free or compressed memory and responds; compare with 7,566 MB used before |
| Host (Mac) | `df -h /` | free disk still above 20 GB |
