# Lab 5 — Encoding forensics

Pair: Solo | Driver first half: Samira Aubakirova  
Machine: lab01ubuntu (UTM Ubuntu 24.04) & macOS  
Date: 2026-11-12  

## What I did

```bash
cd ~/labs/lab05-encoding
sha256sum evidence/staff-broken.csv
file evidence/staff-broken.csv | tee evidence/file-before.txt
file -e csv evidence/staff-broken.csv | tee -a evidence/file-before.txt
xxd -l 16 evidence/staff-broken.csv | tee evidence/hexdump-before.txt
```

```text
919027f1f8c71d85cba3f12aad0f6c927d4e0450cfb2a2d63378a69f14cab270  evidence/staff-broken.csv
evidence/staff-broken.csv: Non-ISO extended-ASCII text, with CRLF line terminators
evidence/staff-broken.csv: Non-ISO extended-ASCII text, with CRLF line terminators
00000000: d2e0 e12e b93b d4c8 ce3b c4ee ebe6 edee  .....;...;......
```

```bash
iconv -f RK1048 -t UTF-8 evidence/staff-broken.csv | head -n 4
```

```text
Таб.№;ФИО;Должность;Отдел;E-mail;Дата приёма
1001;Қасымова Әлия Серікқызы;Главный бухгалтер;Бухгалтерия;a.kassymova@tumar-office.kz;12.03.2019
1002;Иванов Сергей Петрович;Директор по продажам;Отдел продаж;s.ivanov@tumar-office.kz;01.09.2017
1003;Нұрланов Ғалымжан Ерланұлы;Веб-разработчик;ИТ-отдел;g.nurlanov@tumar-office.kz;15.06.2022
```

```bash
bash scripts/fix.sh
```

```text
evidence/staff-fixed.csv: Unicode text, UTF-8 (with BOM) text
00000000: efbb bfd0 a2d0 b0d0 b12e e284 963b d0a4  .............;..
```

```bash
cd ~/labs/lab05-encoding/evidence
file staff-fixed.csv | tee file-after.txt
file -e csv staff-fixed.csv | tee -a file-after.txt
sha256sum staff-broken.csv
```

```text
staff-fixed.csv: Unicode text, UTF-8 (with BOM) text
staff-fixed.csv: Unicode text, UTF-8 (with BOM) text
919027f1f8c71d85cba3f12aad0f6c927d4e0450cfb2a2d63378a69f14cab270  staff-broken.csv
```

## Result

- Identified the source encoding of `staff-broken.csv` as **RK1048** (СТ РК 1048-2002) with `CRLF` line endings (`0x0D 0x0A`) and no UTF-8 BOM (first 16 bytes: `d2 e0 e1 2e b9 3b d4 c8 ce 3b c4 ee eb e6 ed ee`, decoding to `Таб.№;ФИО;Должно`).
- Automated remediation in [`scripts/fix.sh`](scripts/fix.sh) by transcoding `RK1048 -> UTF-8` via `iconv`, prepending a UTF-8 BOM (`EF BB BF`), and normalizing line endings from `CRLF` to `LF`.
- Confirmed `staff-fixed.csv` is detected as `Unicode text, UTF-8 (with BOM) text` with LF endings, renders Cyrillic and Kazakh characters cleanly across terminal (`cat`), Excel, and spreadsheet viewers, and leaves `staff-broken.csv` untouched (`sha256sum` unchanged).

## What did not work the first time

- **Assuming CP1251 (`Windows-1251`) as the source encoding:** Initially testing standard Russian Windows-1251 decoding corrupted Kazakh-specific Cyrillic letters (`Ә`, `Ғ`, `Қ`, `Ң`, `Ө`, `Ұ`, `Ү`, `І`). Checking the regional encoding list (`iconv -l`) revealed **RK1048** (Kazakhstan national 8-bit standard), which decoded both Russian and Kazakh names accurately.
- **Opening UTF-8 CSV without BOM in Excel:** Converting strictly to UTF-8 without a Byte Order Mark (`EF BB BF`) still caused Excel to misinterpret the encoding on double-click; prepending `EF BB BF` in `scripts/fix.sh` resolved automatic UTF-8 detection.

## Evidence

- [`evidence/staff-broken.csv`](evidence/staff-broken.csv) — original unmodified input file in RK1048 encoding with CRLF line endings (SHA-256: `919027f1f8c71d85cba3f12aad0f6c927d4e0450cfb2a2d63378a69f14cab270`).
- [`evidence/staff-fixed.csv`](evidence/staff-fixed.csv) — remediated output file in UTF-8 with BOM (`EF BB BF`) and Unix LF line endings.
- [`evidence/file-before.txt`](evidence/file-before.txt) — `file` and `file -e csv` output showing `Non-ISO extended-ASCII text, with CRLF line terminators` before conversion.
- [`evidence/file-after.txt`](evidence/file-after.txt) — `file` and `file -e csv` output confirming `Unicode text, UTF-8 (with BOM) text` after conversion.
- [`evidence/hexdump-before.txt`](evidence/hexdump-before.txt) — `xxd -l 16` hex dump of the first 16 bytes of `staff-broken.csv` proving single-byte RK1048 encoding and absence of BOM.
- [`evidence/opens-correctly.png`](evidence/opens-correctly.png) — side-by-side screenshot proving readable Russian and Kazakh text in Excel and terminal (`cat`).
