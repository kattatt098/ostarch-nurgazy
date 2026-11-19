# Task 1 — Inventory and interpret

1a. The computer in front of me

| Figure | This computer | Command it came from |
|---|---|---|
| CPU model | Apple M1 | `sysctl -n machdep.cpu.brand_string` |
| Cores / logical processors | 8 / 8 (no Hyper-Threading on M1) | `sysctl hw.physicalcpu hw.logicalcpu` |
| Hardware virtualization | supported and available (`kern.hv_support: 1`); on Apple Silicon there is no separate firmware switch | `sysctl kern.hv_support` |
| Memory: total / available now | 8 GiB (8,589,934,592 bytes) / about 70 MB unused; 7,566 MB used, of which 3,425 MB is compressed | `sysctl hw.memsize`, `top -l 1 \| grep PhysMem` |
| System disk: size, free, SSD or HDD | container 245.1 GB, 68.0 GB free; SSD: yes | `diskutil info /`, `df -h /` |
| Partition style of the system disk | GPT (GUID_partition_scheme) | `diskutil list` |
| OS and version | macOS 15.2 (build 24C101) | `sw_vers` |

Output of every command: `evidence/task1.txt`.

 1b. A machine I have never seen: thin-and-light laptop

My guests (from my build log): Ubuntu Server 2 vCPU, 2 GB RAM, 25 GB disk; Windows 11 2 vCPU, 4 GB RAM, 64 GB disk.

1. Could it run both guests at the same time?

| Resource | Guests need | Laptop has | Left for the host |
|---|---|---|---|
| CPU | 2 + 2 = 4 vCPU | 4 threads | 0 (I assume the host needs at least 2) |
| Memory | 2 + 4 = 6 GB | 8 GB | 2 GB (I assume the host needs at least 3 GB) |
| Disk (nominal) | 25 + 64 = 89 GB | 31 GB free | does not fit |
| Disk (actually used now) | about 25 GB | 31 GB free | about 6 GB, and the disks grow |

Assumption for the host: at least 2 threads, 3 GB RAM and 20 GB disk for the system and updates. Other reasons: Windows 11 itself needs at least 4 GB, the disk is slow eMMC, Windows 11 Home in S mode cannot install a hypervisor, and my guests are ARM64 images while the laptop is x86-64.

Answer: no, it cannot run both guests at these sizes.**

2. What would I upgrade first?

Nothing can be upgraded: memory, eMMC and CPU are soldered. Instead: leave S mode (needed to install a hypervisor), put the VM disks on an external USB SSD, run only one light guest (Ubuntu Server, 1-2 GB RAM, 10-15 GB disk), or use a different machine (at least 16 GB RAM and 256 GB disk).
