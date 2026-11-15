# Ticket 2 — four builds with reasoning

Platform shared by all four (including the Zalman N4 Rev.1 case): AMD AM5, MSI PRO B650M-P, Crucial E100 1 TB NVMe, Deepcool AG400 cooler, Windows 11 Pro (boxed retail licence on a USB stick).
What differs is the part that carries each person's load: CPU tier, RAM, GPU, monitor.
All prices are KZT with VAT, dated 2026-11-14, taken from budget.csv.

---

### Build 1 — accountant

**Load:** large Excel workbooks, accounting software (assumed 1C or similar), bank client in a browser, e-mail and PDFs. Eight hours a day, one person, long uptime.

**Bottleneck:** not the CPU. Two things limit this person: screen area (wide tables, two documents side by side) and the safety of the data on the disk. A 27" 2560x1440 panel has about 78% more pixels than a 1920x1080 one (3.69 M vs 2.07 M), which is the cheapest real productivity gain here. No component I can buy fixes data loss; that needs a backup routine, which is outside this budget (see README, section 2).

**Trade-off:** 16 GB in a single module instead of the 32 GB kit. That meets the hard requirement and saves about 49,000 KZT (139,990 − 90,640), but it is single-channel memory with no headroom. The board has four DIMM slots, so adding a second module later is the upgrade path. No UPS is priced.

**Total:** 523,733 KZT (budget.csv, rows 2–10)

---

### Build 2 — sales manager

**Load:** CRM in a browser with many tabs, Excel or Sheets, messengers, and several hours of video calls a day, sometimes with screen sharing.

**Bottleneck:** memory. Every tab and the call client hold RAM; when it runs out Windows pages to the SSD and the call stutters in the middle of a customer conversation. I have not measured this person's real peak. The requester should send one Task Manager screenshot taken during a busy hour; I treat 16 GB as too tight for this role until that screenshot says otherwise.

**Trade-off:** 32 GB (2x16 GB kit, ~49,000 KZT more than a single 16 GB module) and a plain 24" monitor instead of a 27" one. No discrete GPU, no second screen. Webcam and headset are not priced. The CPU is a Ryzen 5 7600, but the saving over the Ryzen 7 7700 is only about 2,600 KZT (94,936 − 92,300), so this is a small decision, not a real saving: if the director wants uniform CPUs, buying the 7700 here costs almost nothing.

**Total:** 574,563 KZT (budget.csv, rows 11–19)

---

### Build 3 — web developer

**Load:** IDE, several browser profiles with dev tools open, Docker or WSL2 containers or virtual machines, a local database, and repeated build and test runs. Builds are multi-threaded; `node_modules`-style folders mean lots of small files.

**Bottleneck:** memory first (containers + IDE + browser together), then core count during builds, then SSD behaviour on many small files.

**Trade-off:** Ryzen 7 7700 (8 cores, 65 W) instead of anything bigger; 32 GB with two free DIMM slots for 64 GB later; a 27" 2560x1440 monitor, one screen only; no discrete GPU (the integrated one drives the monitor). The CPU is OEM, so the warranty is the reseller's (12 months on the listing), not AMD's boxed warranty.

**Open question:** I assumed Windows 11 Pro with WSL2 so that the whole office runs one OS. A Linux workstation would be licensed too, at zero cost, but it makes the person's machine the odd one out for support. I would ask.

**Total:** 602,848 KZT (budget.csv, rows 20–28)

---

### Build 4 — designer

**Load:** Photoshop, Illustrator, Figma; large layered files; possibly occasional video or 3D. This person judges colour for a living.

**Bottleneck:** the monitor first, because it is the measuring instrument. Then RAM (big layered files), then GPU and VRAM.

**Trade-off:** I put 219,000 KZT into a colour-accurate 27" 2560x1440 panel (ASUS ProArt PA278CV; check the sRGB coverage and calibration claims on asus.com before ordering) rather than a 4K wide-gamut ProArt, which was listed at about 315,000 KZT on the same marketplace (+96,000). If the designer prepares work for print, the answer changes and I would revisit it. The CPU is the Ryzen 7 7700. The GPU is an RTX 5050 with 8 GB of VRAM, the cheapest card that is clearly better than integrated graphics; 8 GB is the part most likely to age first. 700 W PSU instead of 550 W for headroom with the card.

**Total:** 914,983 KZT (budget.csv, rows 29–38)

---

## Sum

| Machine | KZT |
|---|---:|
| Accountant | 523,733 |
| Sales manager | 574,563 |
| Web developer | 602,848 |
| Designer | 914,983 |
| **Four machines with monitors** | **2,616,127** |
| Budget | 3,000,000 |
| Left over | 383,873 (12.8%) |

The leftover is not spare money. It is the reserve for keyboards and mice, a webcam, UPS units, a backup drive, the office-software licences, and for DDR5 prices moving before the order is placed.
