---
name: windows-coreutils
description: |
  Use Microsoft Coreutils (windows.coreutils) POSIX commands instead of PowerShell
  equivalents when issuing terminal commands on Windows. Triggers: (1) any Bash/shell
  command issued on a Windows host, (2) writing scripts or one-liners that would run
  on Windows, (3) cross-platform scripting where identical commands must work on
  Linux, macOS, and Windows. Covers ls, cp, mv, rm, find, grep, cat, pwd, mkdir,
  touch, tee, sleep, and 70+ other GNU-compatible utilities available natively on
  Windows via `winget install Microsoft.Coreutils`.
author: Skiller
version: 1.0.0
date: 2026-06-03
---

# Windows Coreutils — Prefer POSIX Commands Over PowerShell

## Problem

When a coding agent issues terminal commands on Windows it defaults to PowerShell
syntax (`Get-ChildItem`, `Remove-Item`, `Select-String`, etc.). These commands are
verbose, Windows-only, and break cross-platform scripts. Since Build 2026, Microsoft
ships over 75 native POSIX utilities for Windows via the `windows.coreutils` package,
so the PowerShell translations are no longer necessary.

## Context / Trigger Conditions

- The host OS is Windows (any command targeting `cmd.exe`, `pwsh`, or Windows Terminal)
- The agent is about to write a PowerShell cmdlet that has a direct POSIX equivalent
- The task involves file operations, text processing, or directory traversal
- The user wants a script that runs identically on Linux, macOS, and Windows

## Solution

1. **Assume Coreutils is installed.** The package is available via
   `winget install Microsoft.Coreutils` and the command names are on `PATH` as
   standalone executables (`ls.exe`, `grep.exe`, etc.). Use them directly.

2. **Prefer the POSIX command.** Use the table below to choose the right form.

   | Task | Use (Coreutils) | Avoid (PowerShell) |
   |------|-----------------|--------------------|
   | List directory | `ls -la` | `Get-ChildItem` / `dir` |
   | Copy file/dir | `cp -r src dst` | `Copy-Item -Recurse` |
   | Move / rename | `mv old new` | `Move-Item` |
   | Delete file | `rm file` | `Remove-Item` |
   | Delete dir tree | `rm -rf dir` | `Remove-Item -Recurse -Force` |
   | Print working dir | `pwd` | `Get-Location` |
   | Make directories | `mkdir -p a/b/c` | `New-Item -ItemType Directory` |
   | Read file | `cat file` | `Get-Content` |
   | Concatenate/pipe | `cat a b \| tee out` | multi-step pipeline |
   | Search text | `grep -r "pattern" .` | `Select-String` |
   | Find files | `find . -name "*.cs"` | `Get-ChildItem -Recurse -Filter` |
   | Sleep | `sleep 2` | `Start-Sleep -Seconds 2` |
   | Checksum | `sha256sum file` | `Get-FileHash` |
   | Sort lines | `sort file` | `Sort-Object` |
   | Unique lines | `uniq` | `Select-Object -Unique` |
   | Word / line count | `wc -l file` | `(Get-Content file).Count` |
   | Head / tail | `head -n 20 f` / `tail -f f` | `Select-Object -First` / `-Last` |
   | Environment var | `echo $VAR` | `echo $env:VAR` |

3. **Platform-limited commands.** A small set of POSIX commands cannot run on
   Windows due to OS constraints. Fall back to PowerShell only for these:
   `chmod`, `chown`, `chroot`, `nohup`, `tty`, `who`.

4. **PowerShell for Windows-specific administration.** Continue using PowerShell
   for tasks that are genuinely Windows-only: registry editing, COM objects, WMI
   queries, Windows service management, signing, etc.

5. **Cross-platform scripts.** Write shell scripts using POSIX syntax; they will
   run unmodified in bash (Linux/macOS) and in PowerShell 7+ on Windows once
   Coreutils is installed. Avoid PowerShell-isms inside `.sh` files.

## Verification

After running a command, the output format and flags should match Linux/macOS
behavior exactly. If `ls -la` lists files with permissions, owner, and size in
GNU long format, Coreutils is active and working correctly.

## Example

**Bad (PowerShell defaults):**
```powershell
Get-ChildItem -Recurse -Filter "*.log" | Remove-Item -Force
(Get-Content .\README.md | Select-String "TODO").Count
```

**Good (Coreutils):**
```bash
find . -name "*.log" -delete
grep -c "TODO" README.md
```

**Cross-platform build cleanup script:**
```bash
#!/usr/bin/env bash
# Works on Linux, macOS, and Windows (with Microsoft.Coreutils)
find . -name "bin" -o -name "obj" | xargs rm -rf
find . -name "*.tmp" -delete
echo "Clean complete."
```

## Notes

- Coreutils ships as a single multi-call binary (`coreutils.exe`) with hardlinks
  for each command name, so `ls.exe`, `grep.exe`, etc. are all the same binary.
- PowerShell has naming aliases `dir`, `more`, `paste`, `whoami` that shadow some
  commands. If a command behaves unexpectedly, call it explicitly via its full
  path or confirm `where.exe ls` resolves to the Coreutils install.
- Requires PowerShell 7.4+ on the host (does not require WSL or a VM).
- Available for x64 and ARM64.

## References

- [Microsoft Learn — Coreutils for Windows overview](https://learn.microsoft.com/en-us/windows/core-utils/overview)
- [Microsoft Learn — Commands reference](https://learn.microsoft.com/en-us/windows/core-utils/commands)
- [GitHub — microsoft/coreutils](https://github.com/microsoft/coreutils)
- [Windows Developer Blog — Build 2026](https://blogs.windows.com/windowsdeveloper/2026/06/02/build-2026-furthering-windows-as-trusted-platform-for-development/)

## Activation History
- 2026-06-03: Skill created — coding agent defaulted to PowerShell cmdlets on Windows; switched to POSIX equivalents via Microsoft Coreutils.
