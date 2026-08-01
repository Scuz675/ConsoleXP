# Build the TargetGuid ConsoleXP DLL from Linux

You do not need Visual Studio locally. This bundle adds a GitHub Actions workflow that builds `ConsoleXP.dll` on GitHub's Windows Server 2022 runner with Visual Studio 2022.

## What the patch does

It adds this hardware-gated Lua API:

```lua
C_ConsoleXP.TargetGuid(UnitGUID("raid7"))
```

`CheckPermissions(0)` remains in place. Shifty may choose the GUID, but targeting occurs only when you activate the assigned key/controller binding.

## Browser-only method

1. Sign in to GitHub and fork `leoaviana/ConsoleXP`.
2. Download and extract this bundle.
3. In your fork, replace:
   - `src/ConsoleXP/API.cpp`
4. Add:
   - `.github/workflows/build-targetguid.yml`
5. Commit both files to your fork's `main` branch.
6. Open the **Actions** tab.
7. Select **Build ConsoleXP TargetGuid**.
8. Choose **Run workflow**.
9. When the run finishes, open it and download the artifact named:
   - `ConsoleXP-TargetGuid-x86`
10. Extract `ConsoleXP.dll`, back up your current DLL, and replace it with the newly built one.

The artifact also contains `SHA256.txt` so you can identify the exact output you tested.

## Linux terminal method

After creating your fork:

```bash
git clone https://github.com/YOUR_GITHUB_NAME/ConsoleXP.git
cd ConsoleXP

# Copy the two files from this extracted bundle into the clone:
cp /path/to/bundle/src/ConsoleXP/API.cpp src/ConsoleXP/API.cpp
mkdir -p .github/workflows
cp /path/to/bundle/.github/workflows/build-targetguid.yml .github/workflows/

git add src/ConsoleXP/API.cpp .github/workflows/build-targetguid.yml
git commit -m "Add hardware-gated TargetGuid controller targeting"
git push origin main
```

Then run it from GitHub's Actions tab. With GitHub CLI installed, you can instead use:

```bash
gh workflow run build-targetguid.yml
gh run watch
```

After completion, download the artifact with:

```bash
gh run download --name ConsoleXP-TargetGuid-x86
```

## Why this uses GitHub's Windows runner

ConsoleXP hooks 32-bit client functions using MSVC calling conventions such as `__thiscall` and `__fastcall`. A Linux MinGW cross-build may be possible, but using the same MSVC family as the original project reduces the risk of ABI or hook-trampoline differences.

The workflow also fixes the project's developer-specific absolute MinHook paths at build time. The repository already contains the required 32-bit MinHook header and static library.

## First test

Keep the test hardware-triggered. In a party, bind the companion addon action and use `/sxtarget status`. A direct test from a secure key binding can call:

```lua
C_ConsoleXP.TargetGuid(UnitGUID("party1"))
```

Do not test it from an automatic timer or `OnUpdate`; the DLL intentionally keeps ConsoleXP's permission check.
