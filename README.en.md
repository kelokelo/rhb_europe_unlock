# rhb-video-unlock
 
*[Wersja polska: README.md](README.md)*
 
The script modifies the firmware of the **Uconnect 430N (RHB)** head unit from
update package **50.01.01**.
The unit must support 50.01.01. Newer versions change how the disk is handled,
and 50.01.01 does not include the required changes. More info:
https://mygig-disk.com/index-rhb.php
 
- **video while driving** on the front screen;
- **European market**: FM **87.5–108.0 MHz in 100 kHz steps** instead of
  87.7–107.9 in 200 kHz steps (a US unit cannot tune stations on 88.0, 92.6
  etc.).
## ⚠️ Read this first
 
- **No warranty**, MIT licence. Not affiliated with Stellantis, Chrysler or
  the manufacturer of the head unit. You do this **at your own risk**.
- **This repository contains no firmware.** You need the official package
  `RHB-RBZ.50.01.01.zip`. It can be downloaded from
  [mygig-disk.com](https://mygig-disk.com/download-rhb.php), which also
  describes how to install it.
## Which units are supported
 
| Unit | Supported? |
|---|---|
| RHB / RBZ, model years 2011–2015 | **yes** |
| RHB / RBZ, model year 2016, system version **0210 or older** | **yes** |
| RHB / RBZ, model year 2016, system version **0211 or newer** | ⛔ **NO** |
| Newer units (firmware 70.xx.xx and 05.xx.xx) | no (different disk handling) |
 
> ⛔ **System version 0211 or newer:** trying to install 50.01.01 damages the
> unit and it loses access to its hard disk. It can only be repaired in a
> workshop, with the unit removed from the car.
 
Also:
 
- **Working hard disk.** A unit with a faulty disk will not accept the update.
- **Firmware newer than 10.00.11 but older than 33.01.23:** update the unit to
  **33.01.23** first, and only then install 50.01.01.
## What changes
 
| | USA (stock) | Patched |
|---|---|---|
| Video while driving | blocked | **unlocked** |
| FM | 87.7–107.9 MHz, 200 kHz | **87.5–108.0 MHz, 100 kHz** |
| AM | 530–1710 kHz, 10 kHz | 531–1611 kHz, 9 kHz |
| Menu languages | 3 (EN-US, FR-CA, ES) | **6** (EN-UK, FR, ES, DE, IT, NL) |
| RDS | US variant (RBDS) | European RDS, incl. PTY list |
| SiriusXM | present | **gone** (not available in Europe anyway) |
| Radio presets | — | reset on first start |
 
## How it works
 
The unit has two processors:
 
1. **Main SoC** (file `MAIN_APL_CPU1.mbi`). The function that decides whether
   video must be blocked returns its result in register `r2`. At offset
   `0x0B06F8` the bytes `52 F3` → `E2 00` (`mov #0,r2`) make it always return
   "not blocked", i.e. 0.
2. **Audio/CAN MCU** (files `*.aud`). It stores the radio market as an index
   (0 = Europe, 1 = USA, 2 = Japan, 3 = rest of world) and the stock firmware
   **forces USA on every start**. The tuner (FM/AM range and step) depends on
   this index, and the SoC gets the market from the MCU and switches its menus
   to the European version by itself. The script sets the index to 0.
The package has 4 `.aud` variants: `H_I` (CAN I-HS), `H_B` (CAN-B), `HSI` and
`HSB` (the same with SiriusXM). The unit picks its own, so the script patches
all four. Each of the 5 files is checked by SHA-256 before the change and after
writing; the result is byte-identical to the images tested in the car.
 
## What you need
 
- a USB stick (see [Troubleshooting](#troubleshooting));
- [Rufus](https://rufus.ie) (Windows) to format the stick;
- the official `RHB-RBZ.50.01.01.zip` package from
  [mygig-disk.com](https://mygig-disk.com/download-rhb.php);
- [Python](https://www.python.org/downloads/) 3.9 or newer. On Windows, tick
  **"Add python.exe to PATH"** in the installer and leave everything else at
  the defaults;
- the `rhb_europe_unlock.py` script from this repository.
## Step by step (as tested)
 
**0. Power.** Keep the engine running for the whole update. Do not switch the
ignition off or pull the stick while it runs.
 
**1. Stick: MBR partition table, one FAT32 partition.** A stick with a GPT
table is read by the unit, but no update dialog appears. In Rufus, set:
- Boot selection: **Non bootable**;
- Partition scheme: **MBR**;
- File system: **FAT32**.
Click START (**all data on the stick will be erased**). If there are any
`autorun` files on the stick after formatting, delete them.
 
**2. Original first.** Extract the official `RHB-RBZ.50.01.01.zip` to the
root of the stick and flash this **unmodified** package first:
- "Update baseline data files on HDD?" → **YES** (data for the unit's disk);
- application update → **YES**.
**3. Patch the stick with the script.** On the stick with the original
package, copy `rhb_europe_unlock.py` to the root, change to that directory and
run it (Python 3.9+):
 
```text
python rhb_europe_unlock.py
```
 
Easiest way on Windows: open the stick in File Explorer, type `cmd` in the
address bar and press Enter. A command prompt opens directly in the stick's
root directory.
 
The script modifies the files in place, without backups, and verifies their
checksums. If any file is not the original 50.01.01 file, nothing is changed.
 
**4. Flash the patched version.** The unit already runs 50.01.01 and so does
the package — that is fine:
- "Update baseline data files on HDD?" → **NO** (the data is already on the
  disk from step 2 and is not changed);
- application update 50.01.01 → 50.01.01 → **YES**. The unit flashes the
  application ("flash Application software") and the MCU firmware ("flash
  Audio CPU software");
- when it finishes and **offers the same update again** → **CANCEL**. Then
  remove the stick; the unit works.
**Going back to stock:** flash the unmodified 50.01.01 package again.
 
## Troubleshooting
 
- **No update dialog appears.** Check that the stick has an MBR table and a
  FAT32 partition (step 1), and that the package files are in the root
  directory.
- **The unit restarts at around 25% of the installation.** Use a different
  stick. Reported case: a Goodram 16 GB USB 2.0 stick does not work, the same
  capacity in the USB 3.0 version does.
- **`python` is not recognised** in the command prompt. Use `py` instead, or
  reinstall Python with "Add python.exe to PATH" ticked.
 
