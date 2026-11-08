# Windows 10 Minimum Version Upgrade – Actionable Checklist

> **Goal**: Raise the minimum supported OS from Windows 7 (`0x0601`) to Windows 10 (`0x0A00`).
> Verified: VS2026 (v145) and VS2022 (v143) both build clean before changes.
>
> **Strategy**: Implement each section → build with VS2026 → commit & push → move on.

---

## 1. Version Defines – vcxproj Files ✅ (PR #5565)

All preprocessor definitions in every `<PreprocessorDefinitions>` block must change:

| Old                        | New                        |
| -------------------------- | -------------------------- |
| `_WIN32_WINNT=0x601`       | `_WIN32_WINNT=0x0A00`      |
| `WINVER=0x601`             | `WINVER=0x0A00`            |
| `NTDDI_VERSION=0x06010000` | `NTDDI_VERSION=0x0A000000` |

### Notepad3 Core

- [x] `src/Notepad3.vcxproj` – 9 config blocks _(PR #5565)_

### Scintilla

- [x] `scintilla/Scintilla.vcxproj` – 12 config blocks _(PR #5565)_
- [x] `scintilla/ScintillaDLL.vcxproj` – 4 config blocks _(PR #5565)_

### Lexilla

- [x] `lexilla/Lexilla.vcxproj` – 9 config blocks _(PR #5565)_

### Other

- [x] `other_sln/Notepad3DLL.vcxproj` – 4 config blocks _(PR #5565)_

> 🔨 **GATE**: ~~Build with VS2026 → commit → push~~ ✅ Done via PR #5565

---

## 2. Version Defines – Source File Headers ✅ (PR #5565)

PR #5565 updated all headers using `_WIN32_WINNT_WIN10` / `NTDDI_WIN10_RS5` symbolic constants + `#include <sdkddkver.h>`.

### Primary headers

- [x] `src/TypeDefs.h` – updated defines _(PR #5565)_
- [x] `src/TypeDefs.h` – `#if 0` block handled _(PR #5565)_
- [x] `src/resource.h` – updated defines _(PR #5565)_

### Individual source files

- [x] `src/EncodingDetection.cpp` _(PR #5565)_
- [x] `src/Print.cpp` _(PR #5565)_
- [x] `src/PathLib.c` _(PR #5565)_
- [x] `src/Config/Config.cpp` _(PR #5565)_
- [x] `src/crypto/crypto.c` _(PR #5565)_
- [x] `src/crypto/rijndael-api-fst.c` _(PR #5565)_
- [x] `src/ChooseFont/ChooseFont.h` _(PR #5565)_

### Backlog / Test files

- [x] `src/_backlog/AccelKeys.c` _(PR #5565)_
- [x] `test/test_files/StyleLexers/styleLexCPP/Config.cpp` – syntax highlighting test data, not compiled code (left as-is intentionally)

> 🔨 **GATE**: ~~Build with VS2026 → commit → push~~ ✅ Done via PR #5565

---

## 3. Application Manifests – Clean Up Legacy OS GUIDs ✅

### Files to update:

- [x] `res/Notepad3.exe.manifest` – cleaned _(PR #5565)_
- [x] `minipath/res/MiniPath.exe.manifest` – cleaned _(PR #5565)_
- [x] `grepWinNP3/src/compatibility.manifest` – cleaned _(commit 88827d8db)_

> 🔨 **GATE**: ~~Build with VS2026 → commit → push~~ ✅ `88827d8db`

---

## 4. Runtime Version Checks – Dead Code Removal ✅

### `src/Notepad3.c`

- [x] Line 688 – `IsWindowsXPSP3OrGreater() ? 1 : 2` → simplified to `1` _(commit 88827d8db)_
- [x] Lines 1010–1014 – `IsWindows7SP1OrGreater()` guard → updated _(PR #5565)_
- [x] Lines 2829–2839 – `if (!IsWindowsVistaOrGreater())` block removed _(PR #5565)_
- [x] Line 3311 – `!IsWindowsXPSP3OrGreater() &&` removed from condition _(commit 88827d8db)_
- [x] Line 3314 – `if (bProcessed && !IsWindowsXPSP3OrGreater())` block removed _(commit 88827d8db)_
- [x] Line 12112 – `!IsWindowsVistaOrGreater() ||` removed _(PR #5565)_

### `src/Helpers.c`

- [x] Lines 355–361 – `IsProcessElevated()`: `IsWindowsVistaOrGreater()` check removed _(PR #5565)_
- [x] Lines 225–251 – `GetWinVersionString()`: Win7/Win8/Win8.1 branches removed _(PR #5565)_

> 🔨 **GATE**: ~~Build with VS2026 → commit → push~~ ✅ `88827d8db`

---

## 5. Compile-Time Conditionals – Simplified by NTDDI_WIN10 ✅

### `src/Helpers.c` – URL Escape Functions

- [x] Lines 1986–1999 – Removed `#if (NTDDI_VERSION < NTDDI_WIN8)` legacy URL char tables _(commit 88827d8db)_
- [x] Lines 2003–2049 – `UrlEscapeEx()`: Removed `#else` branch, kept Win8+ path _(commit 88827d8db)_
- [x] Lines 2057–2100 – `UrlUnescapeEx()`: Removed `#else` branch, kept Win8+ path _(commit 88827d8db)_

### `src/DarkMode/DarkMode.cpp`

- [x] Lines 58–69 – `#if _WIN32_WINNT < _WIN32_WINNT_WIN8` block removed _(PR #5565)_
- [x] Line 64 – `GetProcAddress` for `SetDefaultDllDirectories` removed _(PR #5565)_

> 🔨 **GATE**: ~~Build with VS2026 → commit → push~~ ✅ `88827d8db`

---

## 6. API Modernizations – Win10 Guaranteed APIs ✅

### `GetTickCount()` → `GetTickCount64()` (avoids 49.7-day rollover)

- [x] `src/Notepad3.c` line 179 – `s_dwAutoScrollStartTick` type: `DWORD` → `ULONGLONG` _(commit 4d9a45805)_
- [x] `src/Notepad3.c` line 2482 – `GetTickCount()` → `GetTickCount64()` _(commit 4d9a45805)_
- [x] `src/Notepad3.c` line 2491 – `GetTickCount()` → `GetTickCount64()` _(commit 4d9a45805)_
- [x] `src/Helpers.c` line 615 – `GetTickCount()` → `GetTickCount64()` _(commit 4d9a45805)_
- [x] `src/Helpers.c` line 617 – `GetTickCount()` → `GetTickCount64()` with `(DWORD)` cast _(commit 4d9a45805)_

### Future Evaluation (not blockers)

- [ ] `RtlAreLongPathsEnabled` – evaluate direct ntdll linkage vs dynamic loading
- [ ] `PathCch*` APIs – evaluate uncommenting `#include <pathcch.h>`
- [ ] `GetSystemMetrics()` → `GetSystemMetricsForDpi()` – evaluate DPI-aware variants

> 🔨 **GATE**: ~~Build with VS2026 → commit → push~~ ✅ `4d9a45805`

---

## 7. Commented-Out / Disabled Legacy Code Cleanup ✅

- [x] `src/Helpers.c` lines 184–222 – `_GetTrueWindowsVersion()` `#if 0` block removed _(commit c9c3dbf5e)_
- [x] `src/Notepad3.c` lines 1251–1253 – Commented-out `ChangeWindowMessageFilter` calls removed _(commit c9c3dbf5e)_
- [x] `src/Notepad3.c` lines 1016–1017 – `SetProcessDpiAwarenessContext` comment replaced with manifest note _(commit c9c3dbf5e)_
- [x] `src/Notepad3.c` line 1091 – `SetThreadDpiAwarenessContext` commented code removed _(commit c9c3dbf5e)_
- [x] `src/PathLib.c` line 67 – Already handled by PR #5565

> 🔨 **GATE**: ~~Build with VS2026 → commit → push~~ ✅ `c9c3dbf5e`

---

## 8. Build Infrastructure ✅

### Build.ps1 – VS2026 Support

- [x] `Build/scripts/Build.ps1` line 26 – vswhere range `[17.0,18.0)` → `[17.0,19.0)` _(commit c9c3dbf5e)_

### build.yml – Version Bump

- [x] `.github/workflows/build.yml` line 43 – `$Major = 6` → `$Major = 7` _(commit bdbc15faa)_

> 🔨 **GATE**: ~~Build with VS2026 → commit → push~~ ✅ `c9c3dbf5e`

---

## 9. Verification ✅

- [x] Build x64 Release with VS2026 – **PASS** (exit code 0, v18.3.2)
- [x] Grep for remaining `0x0601` or `0x06010000` – **clean** (no matches in source)
- [x] Grep for remaining `_WIN32_WINNT_WIN7` or `NTDDI_WIN7` – **clean** (only in SDK header `dlgs.h`)
- [x] Grep for `IsWindowsVistaOrGreater|IsWindows7|IsWindowsXPSP3` – **clean** (no matches)
- [x] Build x64 Release with VS2022 (manual verification)
- [x] Build Win32 Release (manual verification)
- [ ] Smoke-test launch on Windows 10
- [x] Smoke-test launch on Windows 11
- [ ] Verify Dark Mode still works (Win10 1809+)
- [ ] Verify URL copy/escape in hyperlink features
- [ ] Verify toolbar appearance (XP ternary removed)
- [ ] Verify elevated relaunch still works

---

## 10. Documentation ✅

- [x] Created `WinUpgrade.md` in repo root _(commit f2bc77890)_
- [ ] Update any README/docs referencing "Windows 7" as minimum OS requirement
