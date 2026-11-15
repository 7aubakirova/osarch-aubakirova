# Ticket 2 — Office hardware specification

## 1. What was asked

**From:** Daniyar K. (Director)  
**Subject:** computers for the new office  

> We are opening the second office and need four computers — accountant, sales, web developer, designer. Budget 3,000,000 tenge for everything. Everyone says we should just buy the same four machines, is that right? Make sure they last a few years. Monitors included.

---

## 2. What was actually needed

### Where the request is wrong
- **Buying four identical machines is the wrong approach:** At ~750,000 KZT per workstation, a uniform spec over-provisions administrative roles (paying for discrete graphics or high-core CPUs that stay <10% utilized in 1C, Excel, and CRM) while severely under-provisioning the **Designer** (who needs dedicated GPU VRAM and a high-resolution IPS display) and the **Web Developer** (who needs multi-core CPU throughput and 32 GB DDR5 RAM for Docker containers and local builds).

### What the request left out that the job requires
- **Operating system licensing:** Commercial office workstations require genuine **Windows 11 Pro** licenses (~52,000 KZT per seat = 208,000 KZT total), which consume nearly 7% of the budget before buying any hardware.
- **Peripherals, headsets, and UPS protection:** A "complete" workstation requires keyboards, mice, a USB headset for the Sales Manager's daily video calls, and Uninterruptible Power Supplies (UPS) for Almaty office power dips—especially for the Accountant's local/file-server 1C database work.
- **Role-specific baseline minimums:** At least **16 GB RAM** and a **1 TB PCIe 4.0 NVMe SSD** across every machine so none of the four workstations becomes obsolete within 3–4 years.
- **Future upgrade headroom:** Free M.2 slots and standard ATX/mATX power supplies (instead of proprietary OEM pre-built parts) so storage or memory can be expanded later without replacing whole PCs.

### What I would have asked the requester (and what I assumed instead)
1. **Question:** Does the Accountant run a local 1C:Enterprise file database on this PC, or connect to a central 1C server/cloud over VPN?  
   **Assumption:** Assumed a mixed local/network file-base setup; prioritized high random-IOPS NVMe storage, fast single-thread CPU boost (**AMD Ryzen 5 5600GT**), and a dedicated UPS to prevent database corruption during power outages.
2. **Question:** Does the Designer work strictly in 2D/UI/print (Photoshop, Illustrator, Figma) or also heavy 3D rendering/4K video editing?  
   **Assumption:** Assumed commercial 2D branding, UI/UX, photo retouching, and light motion graphics; allocated a **Gainward GeForce RTX 4060 Ti Pegasus 8 GB** (CUDA/Studio acceleration) and a **27" 4K UHD IPS monitor (`Philips 27B2U3601/01`)** rather than an expensive RTX 4070 Ti/4080 workstation GPU.
3. **Question:** Does the new office already have wired Gigabit Ethernet drops at every desk, or will workstations rely on Wi-Fi?  
   **Assumption:** Included built-in Wi-Fi (`MSI B550M PRO-VDH WIFI`) on the Sales Manager's board for flexibility, while assuming wired Cat6 Ethernet for the Accountant and Web Developer.
4. **Question:** Does "3,000,000 tenge for everything" include VAT (НДС) and assembly, or hardware + OS + monitors + peripherals?  
   **Assumption:** Budgeted **2,972,320 KZT** retail (VAT-inclusive prices on Shop.kz / Kaspi.kz in Almaty as of **2026-11-15**), leaving a **27,680 KZT** buffer for cables/delivery.

---

## 3. What I did

1. **Rejected the uniform-PC assumption** and segmented the 3,000,000 KZT budget across four role-specific builds detailed in [`builds.md`](builds.md) and itemized in [`budget.csv`](budget.csv):
   - **Accountant (467,980 KZT):** **AMD Ryzen 5 5600GT** (integrated Radeon, Shop.kz Art. 178966), 16 GB DDR4-3200 (2×8 GB), 1 TB PCIe 4.0 NVMe SSD, 23.8" FHD IPS monitor (`Dell SE2422H`), Windows 11 Pro, UPS + peripherals.
   - **Sales Manager (601,400 KZT):** **AMD Ryzen 5 5600GT**, **32 GB DDR4-3200 (`Geil Pristine`, Art. 197038)** (traded down from Ryzen 7 to double RAM for 40 browser tabs + 4h/day video calls), 1 TB NVMe SSD, 23.8" FHD IPS 100Hz monitor (`LG 24MR400-B`), Windows 11 Pro, wireless peripherals + USB headset.
   - **Web Developer (915,980 KZT):** **AMD Ryzen 7 7700** (8C/16T AM5, Shop.kz Art. 175698), **32 GB DDR5-6400 (`Kingston Fury Beast`, Art. 198357)**, 1 TB PCIe 4.0 NVMe SSD, 27" QHD IPS monitor (`LG 27GR75Q-B`), Windows 11 Pro, mechanical keyboard + mouse (omitted discrete GPU since web dev does not use 3D rendering).
   - **Designer (986,960 KZT):** **Intel Core i5-13400** (10C/16T LGA1700, Shop.kz Art. 167974), **32 GB DDR4-3200 (`Geil Pristine`, Art. 197038)**, **Gainward GeForce RTX 4060 Ti Pegasus 8 GB** (Shop.kz Art. 184429), 1 TB PCIe 4.0 NVMe SSD, **27" 4K UHD IPS (`Philips 27B2U3601/01`, Art. 199051)**, Windows 11 Pro, Logitech MX Master 3S + K380.
   - **Total spent:** **2,972,320 KZT** (**27,680 KZT** under the 3,000,000 KZT budget).
2. **Verified hardware compatibility** on PCPartPicker ([`evidence/pcpartpicker-links.md`](evidence/pcpartpicker-links.md)) and priced every component in KZT from Kazakhstan retailers (**Shop.kz** and **Kaspi.kz**) on **2026-11-15** ([`budget.csv`](budget.csv), [`evidence/vendor-quotes/`](evidence/vendor-quotes/)).
3. **Answered the Director's three operational questions:**
   - **First machine to upgrade in 18 months:** **The Designer's workstation** — adding a secondary **2 TB M.2 NVMe SSD** (~65,000 KZT) into its free M.2 slot (and optionally expanding RAM to 64 GB), because high-DPI `.psd` source files, raw photos, and Adobe scratch caches fill a 1 TB drive far faster than accounting sheets or code repositories.
   - **Pre-built vendor pricing vs. custom builds (gain/loss):** Equivalent pre-built towers (HP Pro Tower / Lenovo ThinkCentre / ASUS ROG / HP Victus) plus matching monitors and Windows 11 Pro cost **~3,420,000 KZT** (+447,680 KZT over budget).
     - **Gain by buying pre-built:** Single-point vendor warranty RMA and zero assembly time.
     - **Loss by buying pre-built:** Exceeds the 3,000,000 KZT budget by ~447,680 KZT (or forces downgrades to 512 GB SSDs / TN panels), plus single-channel RAM sticks, proprietary PSUs/motherboards, and OEM bloatware.
   - **Who will complain within a month and about what:** **The Accountant** — about cramped side-by-side window management when cross-referencing 1C, bank statements, PDFs, and wide Excel sheets on a single 24" 1080p monitor (or 2–5 second 1C file-locking lags if placed on Wi-Fi instead of Gigabit Ethernet). Immediate mitigation: enforce wired Gigabit Ethernet on Day 1 and use the remaining 27,680 KZT buffer plus ~35,000 KZT from Q1 IT OPEX to add a second 24" IPS display if needed.

---

## 4. Evidence

- [`builds.md`](builds.md) — detailed load, bottleneck, and trade-off analysis for each of the four workstations.
- [`budget.csv`](budget.csv) — itemized bill of materials (`role, component, model, price_kzt, source, date`) totaling **2,972,320 KZT** as of **2026-11-15**.
- [`evidence/pcpartpicker-links.md`](evidence/pcpartpicker-links.md) — PCPartPicker compatibility verification notes and build list links for all four configurations.
- [`evidence/vendor-quotes/`](evidence/vendor-quotes/) — captured cart price quotes from **Shop.kz** (Almaty) dated **2026-11-15** verifying KZT pricing and availability.

---

## 5. What I would do differently

- **Switch the Web Developer to a non-RGB DDR5-5600 kit to fund a second monitor for the Accountant from Day 1:** Because consumer DDR5-6400 RGB kits spiked to 372,990 KZT on Shop.kz on 2026-11-15, sourcing a standard OEM/non-RGB 32 GB DDR5-5600 kit (~290,000 KZT) would free up ~83,000 KZT—enough to purchase a second 23.8" IPS monitor (`Dell SE2422H`, 68,000 KZT) for the Accountant upfront and eliminate `Alt+Tab` friction between 1C and Excel on Day 1.
- **Lock in corporate B2B quotes rather than retail cart snapshots:** Consumer retail prices on Kaspi.kz and Shop.kz fluctuate weekly on RAM and NVMe SSDs; for an actual 4-PC office rollout, requesting a formal 7-day B2B invoice with VAT (счёт на оплату с НДС) from a distributor prevents mid-week price jumps during approval.
