# Workstation Builds & Engineering Justification

All prices are sourced in KZT from **Shop.kz (Bely Veter)** and **Kaspi.kz** (Almaty delivery) as of **2026-11-15** and itemized in [`budget.csv`](budget.csv).

---

### Build 1 — Accountant

- **Load:** 1C:Enterprise 8.3 client/file-base operations, multi-sheet Excel workbooks with VLOOKUP/SUMIFS formulas, online banking (`BankClient`), e-Gov / ESF electronic invoicing portals, and PDF/scanner document management.
- **Bottleneck:** **Storage random IOPS and single-thread UI responsiveness**, not multi-core compute or 3D graphics. 1C database indexing and large Excel recalculations stall on slow SATA drives or low-clock CPUs, while total RAM usage rarely exceeds 8–10 GB.
- **Trade-off:** Selected an **AMD Ryzen 5 5600G** with integrated Radeon Graphics and **16 GB DDR4-3200 (2×8 GB dual-channel)** rather than a discrete GPU or 32 GB RAM, paired with a **1 TB PCIe NVMe SSD** and a **24" Full HD IPS** flicker-free monitor. This keeps the build fast and responsive for 4+ years while saving ~250,000 KZT to fund the Developer and Designer workstations.
- **Total:** **448,000 KZT** (`budget.csv`, rows 2–10)

---

### Build 2 — Sales Manager

- **Load:** Cloud CRM (Bitrix24 / amoCRM) in Chrome, 30–40 open customer/product tabs, Excel commercial proposals, WhatsApp/Telegram Desktop, and 4+ hours of Zoom/Google Meet video calls with screen sharing—often simultaneously.
- **Bottleneck:** **Memory capacity (RAM), not CPU.** Chrome with 35–40 active tabs plus an HD video call with screen sharing consumes 11–13 GB of RAM. On a 16 GB machine with Windows 11 and background apps, memory pressure triggers pagefile swapping mid-call, causing audio stutter and browser freezes.
- **Trade-off:** Kept the **AMD Ryzen 5 5600G** (6 cores / 12 threads with integrated graphics) instead of upgrading to a Ryzen 7, and spent the ~65,000 KZT saved on **32 GB DDR4-3200 (2×16 GB)** instead of 16 GB. The CPU stays under 30% utilization in this role; the extra 16 GB of RAM eliminates swap stalls completely.
- **Total:** **544,000 KZT** (`budget.csv`, rows 11–19)

---

### Build 3 — Web Developer

- **Load:** Docker Desktop running 4–6 local containers (PostgreSQL, Redis, Nginx, backend API), Node.js / Vite / Webpack builds, VS Code / JetBrains WebStorm indexing, Git operations, and 30+ browser tabs with DevTools open.
- **Bottleneck:** **Multi-core CPU compile throughput, RAM capacity, and NVMe random I/O.** Running Linux containers via WSL2 alongside a heavy IDE and bundler saturates 6-core CPUs and quickly exhausts 16 GB of RAM.
- **Trade-off:** Invested in an **AMD Ryzen 7 7700** (8 cores / 16 threads, Zen 4), **32 GB DDR5-5600 (2×16 GB)**, a fast **1 TB PCIe 4.0 NVMe SSD** (Kingston KC3000), and a **27" QHD (2560×1440) IPS** monitor for high code/terminal density—while omitting a discrete GPU and relying on the Ryzen 7’s integrated Radeon graphics, since full-stack web development does not use 3D GPU rendering.
- **Total:** **865,000 KZT** (`budget.csv`, rows 20–28)

---

### Build 4 — Designer

- **Load:** Adobe Photoshop (multi-layer high-DPI compositions), Adobe Illustrator, Figma desktop, Adobe Lightroom batch exports, and occasional short motion/promo renders.
- **Bottleneck:** **GPU VRAM / CUDA hardware acceleration and display color accuracy.** Photoshop’s Neural Filters, Canvas Rotate/Zoom, Liquify, and GPU-accelerated effects lag on integrated graphics, while a cheap office TN/VA monitor causes inaccurate brand colors on client deliverables.
- **Trade-off:** Allocated budget to an **NVIDIA GeForce RTX 4060 8 GB** (for reliable CUDA/Studio driver acceleration), **32 GB DDR4-3200 RAM**, and a factory-calibrated **27" QHD (2560×1440) IPS 100% sRGB monitor (ASUS ProArt PA278QV)**, while pairing them with an **Intel Core i5-13400F** (10 cores / 16 threads, no integrated GPU) instead of a Core i7 to keep the build at 1,025,000 KZT.
- **Total:** **1,025,000 KZT** (`budget.csv`, rows 29–38)
