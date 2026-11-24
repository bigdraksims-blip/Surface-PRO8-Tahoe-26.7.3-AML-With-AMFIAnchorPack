# Surface-PRO8-Tahoe-26.7.3-AML-With-AMFIAnchorPack
OpenCore AMFIAnchorPACK &amp; dual-boot-safe SSDT-PNLF-SP8 bundle for Microsoft Surface Pro 8 on macOS Tahoe 26.7.3 (25G342) with WhateverXe (1536 MB Metal acceleration + Darwin-only ACPI backlight).






# Surface Pro 8 macOS Tahoe Graphics & Backlight Pack (`AMFIAnchorPACK` + `SSDT-PNLF-SP8`)

A dual-boot-safe OpenCore add-on bundle for the **Microsoft Surface Pro 8 (11th Gen Intel Tiger Lake / Iris Xe `0x9A49`)** running **macOS 26.7.3 Tahoe** with [WhateverXe](https://github.com).

---

## What's Included

1. **`AMFIAnchorPACK/`**
   * `AMFIAnchor.kext` — Core AMFI plugin required on macOS Tahoe to allow `WhateverXe Graphics` Metal/3D bundles to load.
   * `AMFIAnchorInjector.kext` — Codeless certificate/anchor injector kext (`ExecutablePath` must be left empty).
   * **Result:** Unlocks **full 1536 MB VRAM & 3D Metal acceleration** (translucent Liquid Glass UI) on macOS 26.7.3 Tahoe.

2. **`ACPI/` (`SSDT-PNLF-SP8.aml` & `SSDT-PNLF-SP8.dsl`)**
   * Declares `Device (PNLF)` (`APP0002`, `_UID = 0x13`) specifically on the Surface Pro 8's Tiger Lake iGPU path (`\_SB.PC00.IGPU`).
   * Guarded by `If (_OSI ("Darwin"))` so **only macOS** loads the backlight device—preventing `ACPI_BIOS_ERROR (0xA5)` BSODs when booting Windows through OpenCore.
   * **Result:** Enables internal display backlight power/control (`60 Hz`) safely alongside dual-boot Windows.

---

## Installation (`config.plist`)

### 1. Copy Files to Your EFI
* Copy `SSDT-PNLF-SP8.aml` into `EFI/OC/ACPI/`.
* Copy both `AMFIAnchor.kext` and `AMFIAnchorInjector.kext` into `EFI/OC/Kexts/`.

### 2. `ACPI -> Add`
Enable `SSDT-PNLF-SP8.aml`:
* `Path`: `SSDT-PNLF-SP8.aml`
* `Enabled`: `True`

### 3. `Kernel -> Add` (Exact Load Order Matters!)
> **Important:** Edit `config.plist` manually rather than running OC Clean Snapshot so load order is preserved:
1. **`Lilu.kext`** — Must be at position `0` (top of `Kernel -> Add`).
2. **`AMFIAnchor.kext`**
   * `BundlePath`: `AMFIAnchor.kext`
   * `ExecutablePath`: `Contents/MacOS/AMFIAnchor`
   * `PlistPath`: `Contents/Info.plist`
   * `Enabled`: `True`
3. **`AMFIAnchorInjector.kext`** *(Must load **below** `AMFIAnchor.kext`)*
   * `BundlePath`: `AMFIAnchorInjector.kext`
   * `ExecutablePath`: `""` *(Leave completely blank — codeless injector kext)*
   * `PlistPath`: `Contents/Info.plist`
   * `Enabled`: `True`
4. **`WhateverXe.kext`**

### 4. `NVRAM -> Add -> 7C436110-AB2A-4BBB-A880-FE41995C9F82`
* `csr-active-config`: `<03080000>` *(or `<030A0000>`)*
* `boot-args`: Include `-allow3d`

---

## Current Status & Known Issues

* **Working:**
  * Full **1536 MB 3D Metal acceleration** on macOS 26.7.3 Tahoe via USB-C / HDMI external display.
  * Internal Surface Pro 8 display detects as `2880 x 1920` Retina (`60 Hz`) and powers on its LED backlight cleanly.
  * Dual-boot Windows through OpenCore boots with **zero `ACPI_BIOS_ERROR (0xA5)` BSODs**.
* **Known Issue — Internal LCD Pixels (`Lit Backlight, Black Screen` on `WhateverXe 0.8.5`):**
  * While `SSDT-PNLF-SP8.aml` + `WhateverXe 0.8.5` powers on the internal panel's LED backlight (`60 Hz`), `WhateverXe 0.8.5` currently hits a Tiger Lake **eDP 1.4 optional link-rate (`MAX_LINK_RATE=0`)** link-training bug (`kern_xe.cpp`), leaving the lit LCD screen black until an updated `WhateverXe.kext` (`0.8.6+`) incorporating the accepted fix in [b00t0x/WhateverXe Issue #154](https://github.com/issues/154) is installed alongside this pack.
