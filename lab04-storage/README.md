 Lab 4 — Storage

 Windows

Disk 0: partition style GPT, firmware_type UEFI — because Windows booted in UEFI mode, and UEFI firmware boots from a GPT disk (it needs the EFI System Partition that only GPT layouts carry in this setup).

New disk (Disk 1): partition style GPT — because I wanted it to match the system disk and have no limits that could matter later. A 2 GB disk fits either table, so the choice only matters in other situations: a disk larger than 2 TB (MBR cannot address it), a disk that must be booted from on a UEFI machine (needs GPT), or a disk that needs more than four partitions (MBR allows only four primary partitions; GPT allows 128). GPT also keeps a backup copy of the partition table at the end of the disk and checks it with a CRC, which MBR does not.

Commands used (PowerShell as Administrator):

```powershell
Initialize-Disk -Number 1 -PartitionStyle GPT
New-Partition -DiskNumber 1 -UseMaximumSize -AssignDriveLetter | Format-Volume -FileSystem NTFS -NewFileSystemLabel LAB4
```

Letter E was already in use, so Windows picked a free letter automatically: F.

Output of `Get-Disk` (after):

```
Number FriendlyName     Size         PartitionStyle OperationalStatus
------ ------------     ----         -------------- -----------------
     0 QEMU NVMe Ctrl   68719476736  GPT            Online
     1 QEMU NVMe Ctrl    2147483648  GPT            Online
```

Output of `Get-Partition -DiskNumber 1`:

```
   DiskPath: \\?\scsi#disk&ven_nvme&prod_qemu_nvme_ctrl#4&399fe65&0&000000#{53f56307-b6bf-11d0-94f2-00a0c91efb8b}

PartitionNumber  DriveLetter Offset                                        Size Type
---------------  ----------- ------                                        ---- ----
1                            17408                                     15.98 MB Reserved
2                F           16777216                                   1.98 GB Basic
```

Output of `$env:firmware_type`:

```
UEFI
```

 Linux layout — before

```
NAME                       SIZE TYPE FSTYPE      MOUNTPOINT
sda                        3.6G disk iso9660     
├─sda1                     3.6G part iso9660     
└─sda2                     7.1M part vfat        
vda                         25G disk             
├─vda1                       1G part vfat        /boot/efi
├─vda2                       2G part ext4        /boot
└─vda3                    21.9G part LVM2_member
  └─ubuntu--vg-ubuntu--lv   11G lvm  ext4        /
vdb                          2G disk             
vdc                          2G disk             
vdd                          2G disk             
```

The system disk is `vda` (25 GB, mounted at `/` and `/boot`). `sda` is the installer ISO. The three new 2 GB disks with no partitions and no mount point are `vdb`, `vdc` and `vdd`.

 Linux layout — after the rebuild

```
NAME                       SIZE TYPE  FSTYPE            MOUNTPOINT
sda                        3.6G disk  iso9660           
├─sda1                     3.6G part  iso9660           
└─sda2                     7.1M part  vfat              
vda                         25G disk                    
├─vda1                       1G part  vfat              /boot/efi
├─vda2                       2G part  ext4              /boot
└─vda3                    21.9G part  LVM2_member       
  └─ubuntu--vg-ubuntu--lv   11G lvm   ext4              /
vdb                          2G disk                    
└─vdb1                       2G part  linux_raid_member
  └─md0                      2G raid1 ext4              /mnt/mirror
vdc                          2G disk                    
└─vdc1                       2G part  linux_raid_member
vdd                          2G disk                    
└─vdd1                       2G part                    
  └─md0                      2G raid1 ext4              /mnt/mirror
```

| Role            | Device |
|-----------------|--------|
| Mirror half A   | vdb1   |
| Failed half     | vdc1   |
| Replacement     | vdd1   |

Note: `vdc1` still carries RAID metadata (`linux_raid_member`) but belongs to no array any more.

 The three states of the array

Each state was saved the moment it existed (see `evidence/`):

| State    | File                 | Time (UTC) | Status                          |
|----------|----------------------|------------|---------------------------------|
| Healthy  | array-healthy.txt    | 06:15:56   | `[2/2] [UU]`, clean             |
| Degraded | array-degraded.txt   | 06:19:25   | `[2/1] [U_]`, clean, degraded   |
| Rebuilt  | array-rebuilt.txt    | 06:25:03   | `[2/2] [UU]`, clean             |

Proof that the data survived the failure: after `umount` and `mount` (so the files were read from the surviving disk, not from the cache), `diff checksum-before.txt checksum-after-failure.txt` printed nothing and `MATCH` was shown. `proof.txt` still read: `Lab 4 - NURGAZY - Thu Nov  5 06:18:35 UTC 2026`.

 The question

Your colleague deleted the wrong file from the mirror an hour ago.
Does RAID 1 help? Why?

No, RAID 1 does not help here. A mirror writes every change to both disks at the same moment, so the deletion was copied to both halves immediately, and RAID 1 only protects against the loss of a disk, as in section 7 where the data survived on one disk, not against a human mistake. A backup or a snapshot taken before the deletion and stored independently of the mirror would have let us restore the file.
