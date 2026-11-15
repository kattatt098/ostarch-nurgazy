# Ticket 2 — We are kitting out the new office

## 1. What was asked

> From: Daniyar K. (Director)
> Subject: computers for the new office
>
> We are opening the second office and need four computers — accountant,
> sales, web developer, designer. Budget 3,000,000 tenge for everything.
> Everyone says we should just buy the same four machines, is that right?
> Make sure they last a few years. Monitors included.

## 2. What was actually needed

**Where the request is wrong.** "Just buy the same four machines" is the wrong answer to the question the director really has, which is how to spend 3,000,000 KZT without waste. The four jobs stress four different parts: the accountant needs screen area and data safety, the sales manager needs memory for browser tabs and video calls, the developer needs memory and cores for containers and builds, the designer needs an accurate monitor. Identical machines overspend on parts some roles never use and underspend on the part each role depends on. What can be identical is the platform (CPU, board, case, SSD, cooler, power supply, OS), and I made it identical, because that simplifies spare parts and support. What differs is memory and monitor, which is where the loads differ.

**What the request left out, what I would have asked, and what I assumed instead:**

| Gap | What I would ask | What I assumed |
|---|---|---|
| Real workloads | A Task Manager screenshot at each person's busiest hour | Typical loads for each role (see builds.md). Memory figures are my estimate, not a measurement |
| Where to deliver | City and office address | Almaty. Prices come from shops that dispatch from Almaty or Astana |
| "For everything" | Does it include keyboards, mice, webcam, UPS, software? | Hardware, monitors and the OS only. The small leftover covers keyboards and mice at most |
| Office and business software | Office suite? Accounting software (1C)? | Not included; the budget does not cover licences I cannot identify |
| "Last a few years" | How many? | 4–5 years, with one planned upgrade around 18 months. This is why I chose AM5 (the CPU can be swapped later) and kept free RAM slots |
| Who builds and supports them | Is there an IT person? | Someone assembles and installs; that labour is not priced (see Q2) |
| Backups | Where does the accountant's data live? | It does not live only on that disk. No component fixes this; it needs a backup routine |
| Windows or Linux for the developer | Which does the developer use? | Windows 11 Pro with WSL2 for a single-OS office |
| Designer's output | Screen only or print too? Video or 3D? | Screen only, 2D work. If print, the monitor changes; if video or 3D, a graphics card has to be added |
| Warranty and legality | Invoice, warranty terms? | Licensed OS is required, so I priced only a boxed retail Windows licence from a named listing. Listings at 2,000–20,000 KZT exist on marketplaces; they are not a licence I could defend and I did not use them |

**Result.** Four different machines for 2,907,946 KZT with monitors, 92,054 KZT under budget. The hard requirements (16 GB or more, 1 TB SSD, licensed OS, monitor) are met on every machine. The budget is tight because memory is expensive right now: one 16 GB module cost about 140,000 KZT and a 32 GB kit about 294,000 KZT in the shops I checked.

## 3. What I did

1. **Described each role by load** and wrote down the bottleneck and the trade-off for it (builds.md).
2. **Chose the platform.** AMD AM5 with an MSI PRO B650M-P board: DDR5, a socket still being sold, so the CPU can be replaced later. Alternative considered: Intel LGA1700, cheaper to find as pre-built, but that socket is at the end of its life, which conflicts with "last a few years".
3. **Priced every part from Kazakhstani shops** and recorded the shop, the link and the date in budget.csv, with a screenshot of each page in evidence/vendor-quotes/. DDR5 memory showed the widest spread: a 2x16 GB kit was 293,990 KZT at one retailer, 309,990 at another and 372,990 for a faster Kingston kit on 14 November. That is why every price is dated.
4. **Ryzen 7 7700 OEM for all four.** On 14 November it was listed at 91,726 KZT, slightly cheaper than the Ryzen 5 7600 OEM at 92,670, so the weaker CPU would have cost more for less. I chose OEM plus a separate AG400 cooler over boxed CPUs. Cost: reseller warranty instead of the manufacturer's.
5. **Windows 11 Pro, boxed retail, USB, KZ-only (Russian)** at 78,000 KZT. Alternatives: the OEM disc edition (about 84,000 KZT in the listings I saw, tied to the machine and shipped on a DVD that these cases have no drive for), and Home edition (no BitLocker or domain join). The retail box is cheaper in these listings, transferable to a new PC when the old one is retired, and comes on a USB stick.
6. **Spent the money where the load is.** Accountant: 16 GB and a 27" QHD monitor. Sales: 32 GB. Developer: 32 GB and a 27" QHD monitor. Designer: 32 GB and a colour-accurate monitor, with no graphics card (about 204,000 KZT, which the budget could not carry).
7. **Checked compatibility by hand** (evidence/pcpartpicker-links.md): socket, DDR5, M.2, video outputs, cooler height and power-supply length against the case.
8. **Summed the four builds** against 3,000,000 KZT. The reserve is 92,054 KZT and is already spoken for (see builds.md).

### Answers to the director's three questions

**1. Which of the four machines will I upgrade first, eighteen months from now, and why?**
The accountant's. It is the one built at the minimum on memory: 16 GB in a single module, which meets the requirement today but has no headroom, and software memory use only grows. The fix is a second 16 GB module (about 140,000 KZT at today's price; the board has four DIMM slots), which also turns single-channel memory into dual-channel. Memory is also the component whose price moved most between the first and last shops I checked, so leaving the slot free and buying later is a deliberate choice. The designer's graphics card is the second candidate if the work turns out to include video or 3D.

**2. Price the same four as pre-built machines. What does the company gain and lose by not building them?**
Closest pre-built I could price in Kazakhstan: HP Pro Tower 400 G9, Core i5-14500, 16 GB DDR5, 1 TB SSD, Windows 11 Pro, 3-year warranty in the model name, keyboard and mouse included: 774,928 KZT (Satu.kz listing; I also saw the same family at roughly 720,000–740,000 KZT in other listings, but those come and go, so I rely on this one). Four of them are 3,099,712 KZT. Adding the same four monitors (413,230 KZT) gives 3,512,942 KZT. That is 512,942 KZT over budget, 604,996 KZT more than the custom builds, and every machine would still have 16 GB and integrated graphics, so the sales manager, the developer and the designer would be underpowered. I did not find a verified pre-built with 32 GB inside the budget.
*Gains from pre-built:* one vendor and one invoice, one warranty (the model name says 3 years; I did not confirm on-site or return-to-base), Windows pre-installed and tested, no assembly time, business-grade models.
*Losses:* about 0.6 M KZT more for weaker machines, fixed specs (a 260 W supply rules out a discrete graphics card), no free choice of RAM or monitor, and one spec for four different jobs, which is the director's original mistake in a different form.
*What the custom price leaves out:* assembly and Windows installation time (my estimate: half a day per machine), keyboards and mice, and the fact that each part has its own warranty and its own shop.

**3. One of these four will complain about performance within a month. Who, and about what?**
The designer, about anything that uses the graphics card: video, 3D and GPU-accelerated filters, and exporting large files, feeling slower than the person expects. I spent the budget on the monitor and 32 GB and left out the graphics card, and designers compare against whatever they used before. The accountant's likely complaint is that the accounting database or shared Excel files are slow, which is usually the network or the shared storage, not this PC; the sales manager's video call problems are more likely Wi-Fi or bandwidth.

## 4. Evidence

- `budget.csv` — every component, model, price in KZT, shop and link, and the date it was taken. This proves the totals and that the vendors are in Kazakhstan.
- `builds.md` — per-machine load, bottleneck, trade-off and totals, with row references into budget.csv.
- `evidence/vendor-quotes/` — a screenshot of every priced page, taken on 14 November 2026. Each file is named after the budget.csv row it supports; the README there maps files to rows.
- `evidence/pcpartpicker-links.md` — the hand compatibility check for every part pairing.

## 5. What I would do differently

My first price list was wrong in several places, and the screenshots showed it: I had read the price of a single 16 GB memory module as the price of a 32 GB kit, taken a graphics-card price from an offer that was no longer cheapest, and opened an SSD and a Windows page for the wrong variant. Next time I would open every product page and take the screenshot before the price goes into the spreadsheet, not after. I would also ask the shops for a corporate quote, because a company orders from an invoice, not from a catalogue; measure real workloads (Task Manager from each future user) instead of estimating memory use; and physically check cooler and power-supply clearance in the Zalman N4 before ordering.
