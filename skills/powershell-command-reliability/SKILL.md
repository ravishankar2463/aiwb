---
name: powershell-command-reliability
description: Use when writing or running PowerShell commands, using PowerShell through an MCP or terminal tool, or diagnosing a failed command to avoid blind retries and wasted execution.
---

# PowerShell command reliability

Make each command attempt informative. Do not repeat a failed command unchanged.

## Before running a command

1. Identify the actual execution tool and shell. Do not assume PowerShell, CMD, or another shell has the same syntax. In Windows, `powershell.exe` is usually Windows PowerShell 5.1 and `pwsh.exe` is PowerShell 7+. When relevant, check:

   ```powershell
   $PSVersionTable.PSVersion
   $PSVersionTable.PSEdition
   $Host.Name
   $PWD.ProviderPath
   ```

2. Use syntax supported by the confirmed host. PowerShell 7 supports pipeline chain operators `&&` and `||`; Windows PowerShell 5.1 does not. If a command may run on either, use separate statements and explicit `if` checks, or verify which executable the tool launches.
3. Treat each tool invocation as a possibly fresh process. Do not rely on a previous call's current directory, variables, aliases, or environment changes unless that tool explicitly preserves its session.
4. Keep the first attempt small and bounded. Quote paths with spaces, prefer `-LiteralPath` for filesystem paths, and use `Join-Path` to build paths.
5. For a potentially destructive or machine-wide operation, inspect the target and use the tool's preview mode first. Do not retry a destructive operation just because its output was unclear.
6. If the command must work in both Windows PowerShell 5.1 and PowerShell 7, avoid version-specific syntax and test any native-command argument quoting on both hosts when arguments contain spaces, quotes, or backslashes.

## When a command fails

1. Stop before retrying. Preserve the exact command and the complete error/output available.
2. Classify the failure: wrong shell or unsupported syntax, command not found, path/working-directory issue, permissions, PowerShell error, native executable exit code, timeout, tool/transport error, or output truncation.
3. Check only the evidence needed for that class:
   - Resolve a missing command with `Get-Command <name> -ErrorAction SilentlyContinue`.
   - For a missing path, inspect the exact quoted path and current directory.
   - For cmdlet errors that must stop the command, use `-ErrorAction Stop` on that operation; do not hide failures with broad catches.
   - For a native executable, capture and check `$LASTEXITCODE` immediately after it runs on both hosts. Do not assume a printed result or empty output means success. PowerShell 7.3+ can optionally turn native nonzero exit codes into PowerShell errors with `$PSNativeCommandUseErrorActionPreference`; that preference is off by default and may not exist in earlier versions, so do not rely on it instead of checking the exit code.
   - If output was saved to a temporary file or truncated, search or read that file in bounded ranges instead of rerunning the original command.
   - If the tool itself failed before the command ran, fix the tool invocation or session rather than changing the PowerShell command.
4. State a concrete cause or testable hypothesis. Change the command only to address that cause, then run the smallest check that proves it.
5. If the evidence does not identify a cause, ask for the missing detail or report the blocker. Do not cycle through syntax variants or repeat network/install/admin operations speculatively.
6. After a successful run, verify the requested effect with a focused read-only check when possible.

## Useful checks

```powershell
# Confirm a command resolves
Get-Command git -ErrorAction SilentlyContinue

# Check a native program's exit code immediately
git --version
$nativeExitCode = $LASTEXITCODE
if ($nativeExitCode -ne 0) {
    throw "git failed with exit code $nativeExitCode"
}

# Run a cmdlet and make its non-terminating error stop this operation
Get-Item -LiteralPath 'C:\path\with spaces\file.txt' -ErrorAction Stop
```

`$LASTEXITCODE` is for native programs. PowerShell cmdlets report errors through PowerShell's error stream; use the operation's `-ErrorAction Stop` when a script must stop on that error. Check the exit code before running another native command, because it describes the most recently run native program.

PowerShell 7 also changed how `$?` is preserved inside parentheses, subexpressions, and array expressions compared with versions before 7. Prefer capturing the native exit code directly rather than relying on `$?` after wrapping a command.

## References

- [PowerShell automatic variables (`$?` and `$LASTEXITCODE`)](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_automatic_variables?view=powershell-7.6)
- [PowerShell preference variables (`$ErrorActionPreference` and `-ErrorAction`)](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_preference_variables?view=powershell-7.6)
- [PowerShell pipeline chain operators](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_pipeline_chain_operators?view=powershell-7.6)
- [PowerShell parsing and native-command argument passing](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_parsing?view=powershell-7.6)
