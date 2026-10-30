Upgrade Verdict — MacBook Air (M1, 2020)
1. CPU — Apple M1
Current configuration: Apple M1, 8-core CPU.
Upgrade verdict: Not user-upgradeable.
Reason: The CPU is integrated into Apple's M1 System on a Chip (SoC). It is not installed as a separate removable processor. Replacing it would require replacing the entire logic board.
Evidence:
- Command: `system_profiler SPHardwareDataType`
- Field: Chip — Apple M1; Total Number of Cores — 8.
- Service documentation: Apple Repair Manual, Logic Board assembly, pp. 6–7.
2. RAM — 8 GB
Current configuration: 8 GB unified memory.
Upgrade verdict: Not user-upgradeable.
Reason: Unified memory is integrated into the M1 chip package. There are no removable RAM modules or standard memory slots. Increasing memory requires choosing a different factory configuration when purchasing the Mac.
Evidence:
- Command: `system_profiler SPMemoryDataType` — Memory: 8 GB; Type: LPDDR4.
- Command: `sysctl hw.memsize` — hw.memsize: 8589934592 bytes.
- Service documentation: Apple Repair Manual, Logic Board assembly, pp. 6–7.
3. SSD — APPLE SSD AP0256Q
Current configuration: The MacBook Air has an internal Apple SSD with a reported capacity of 251.0 GB (approximately 256 GB nominal).

Command-line evidence: The macOS command `diskutil info disk0` reports:
- Device / Media Name: APPLE SSD AP0256Q
- Protocol: Apple Fabric
- Solid State: Yes
- Device Location: Internal
- Disk Size: 251.0 GB

Service manual evidence: The Apple Repair Manual (MacBook Air M1, 2020), pages 6–7, shows the internal hardware layout and the logic board assembly. It does not identify a standard removable M.2 or SATA SSD module.

Upgrade verdict: The SSD is not designed for a standard user-performed upgrade. Unlike a laptop with a removable M.2 drive, this MacBook does not provide a conventional SSD slot for a simple storage replacement. Storage-related repair requires specialized service.

Conclusion: The SSD is not user-upgradeable through a standard replacement procedure.
