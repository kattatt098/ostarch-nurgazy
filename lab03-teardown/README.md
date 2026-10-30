Lab 03 — Teardown Without a Screwdriver

1. Device Identification

Device: MacBook Air (M1, 2020)  
Model Identifier: MacBookAir10,1  
Model Number: MGN63RU/A  
Operating System: macOS 15.2

The exact model was identified using `system_profiler SPHardwareDataType`.

Manufacturer: Apple  
Service documentation: Apple Repair Manual for MacBook Air (M1, 2020)

2. Hardware Components

The exploded view and component descriptions are based on the Apple Repair Manual for MacBook Air (M1, 2020), pages 6–7.

The component analysis is available in [components.md](components.md).

The annotated exploded view is stored in:

`evidence/manual-figures/exploded-view.png`

Each component is described by its function, connections, and limitations.

3. Hardware Upgrade Verdict

| Component | Current configuration | Upgrade verdict |
|---|---|---|
| CPU | Apple M1, 8-core | Not user-upgradeable |
| RAM | 8 GB unified memory | Not user-upgradeable |
| SSD | APPLE SSD AP0256Q, 256 GB nominal | Not a standard user upgrade |

The CPU and unified memory are integrated into the Apple M1 platform. They cannot be upgraded separately using standard removable components.

The internal SSD is not a standard M.2 or SATA user-upgrade module. Storage service requires specialized procedures.

Detailed reasoning is available in [upgrade-verdict.md](upgrade-verdict.md).

4. Operating System Hardware Comparison

This comparison uses the physical MacBook host running macOS and an Ubuntu virtual machine. Ubuntu values describe virtual hardware assigned to the VM, not the physical MacBook components.

| Hardware fact | macOS — MacBook host (Command — Field — Value) | Ubuntu — Virtual machine (Command — Field — Value) |
|---|---|---|
| CPU model | `sysctl -n machdep.cpu.brand_string` — Output: Apple M1 | `lscpu` — Vendor ID: Apple; exact model not reported |
| Architecture | `system_profiler SPHardwareDataType` — Chip: Apple M1 (ARM architecture) | `lscpu` — Architecture: aarch64 |
| Physical / virtual CPUs | `sysctl hw.physicalcpu` — hw.physicalcpu: 8 | `lscpu` — CPU(s): 2 (virtual CPUs) |
| Logical CPUs | `sysctl hw.logicalcpu` — hw.logicalcpu: 8 | `lscpu` — CPU(s): 2 |
| Total memory | `system_profiler SPHardwareDataType` — Memory: 8 GB | `free -h` — total: approximately 1.9 GiB |
| Memory type | `system_profiler SPMemoryDataType` — Type: LPDDR4 | `sudo dmidecode -t memory` — Type: DRAM (virtual SMBIOS) |
| Memory speed | `system_profiler SPMemoryDataType` — Speed: not reported | `sudo dmidecode -t memory` — Speed: Unknown |
| Disk type | `diskutil info disk0` — Solid State: Yes; Internal | `lsblk -o NAME,SIZE,TYPE,MODEL` — TYPE: disk; virtual disk |
| Disk model | `diskutil info disk0` — Device / Media Name: APPLE SSD AP0256Q | `lsblk -o NAME,SIZE,TYPE,MODEL` — MODEL: Virtual Disk (/dev/sda); /dev/vda model not reported |
| Disk interface | `diskutil info disk0` — Protocol: Apple Fabric | `lsblk -o NAME,SIZE,TYPE,MODEL` — interface: not reported |
| Disk capacity | `diskutil info disk0` — Disk Size: 251.0 GB | `sudo fdisk -l` — /dev/vda: 25 GiB; /dev/sda: approximately 3.56 GiB |
| Motherboard | `system_profiler SPHardwareDataType` — Model Identifier: MacBookAir10,1; Apple M1 platform | `sudo dmidecode -t baseboard` — no populated baseboard model information |

Differences Between the Systems

macOS and Ubuntu provide hardware information in different ways because they use different system tools and run in different environments.

macOS runs directly on the physical MacBook Air and reports its actual hardware specifications, including the Apple M1 processor, 8 GB of unified memory, and internal SSD.

Ubuntu runs inside a VirtualBox virtual machine. It reports the virtual resources assigned to it: 2 virtual CPUs, 2 GB of RAM, and a 25 GB virtual disk. These values describe the virtual machine, not the physical MacBook hardware.

Some details, such as the motherboard model and memory speed, may not be available in Ubuntu because the virtual machine does not expose all physical hardware information to the guest operating system.

5. Evidence Files
The `evidence/` directory contains the following files:

- [macos-specs.txt](evidence/macos-specs.txt)  macOS hardware commands and recorded results.
- [linux-specs.txt](evidence/linux-specs.txt)  Ubuntu virtual machine commands and recorded results.
- [exploded-view.png](evidence/manual-figures/exploded-view.png)  Apple service manual exploded view.

6. Conclusion

The MacBook Air (M1, 2020) has limited user upgradeability. Its CPU and unified memory are integrated into the Apple M1 platform, and its internal storage is not designed for a standard user-performed upgrade.

The comparison also demonstrates that virtual machine specifications differ from physical host specifications. Hardware information must always be interpreted according to the operating system and environment from which it was collected.