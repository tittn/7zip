# 7-Zip fork: archive-named temp folders

This is a fork of [7-Zip](https://github.com/ip7z/7zip) ([7-zip.org](https://7-zip.org)) with one small change to the **7-Zip File Manager** (`7zFM.exe`).

## What's different

When you open a file from inside an archive (for example double-clicking `readme.txt` inside `logs.zip`), 7-Zip extracts it to a temporary folder and opens it from there.

| | Temp path for `readme.txt` from `logs.zip` |
|---|---|
| Official 7-Zip | `%TEMP%\7zO1A2B3C4D\readme.txt` (random name, new every time) |
| This fork | `%TEMP%\7zO_logs.zip\readme.txt` (named after the archive, always the same) |

Why it helps:

- **You can see where a file came from.** The editor's title bar and path show the archive name.
- **Editors can reload instead of opening a duplicate.** Opening the same file again, from any 7-Zip window, overwrites the same path. Editors such as Notepad++ notice the change and offer to reload the tab, instead of opening a second copy from a new random folder.

Details:

- The archive name is cleaned of characters Windows doesn't allow in paths and cut to 64 characters.
- For nested archives, the innermost archive's name is used.
- An earlier copy of the file in that folder is replaced without asking.
- Files that are not opened from an archive still use the original random folder name.
- Everything else is unchanged from upstream 7-Zip.

The whole change is in [`CPP/7zip/UI/FileManager/PanelItemOpen.cpp`](CPP/7zip/UI/FileManager/PanelItemOpen.cpp), in `CPanel::OpenItemInArchive` (look for `// Fork change`).

### Known limitations

- **Same-named files collide.** Two files with the same name in different subfolders of one archive (e.g. `a\log.txt` and `b\log.txt`) share one temp path. Opening one replaces the other.
- **Early cleanup.** When a 7-Zip window closes, it deletes its unedited temp copy. If another window opened the same file from the same archive, the editor may then report the file as deleted. Opening it again brings it back.
- **No reload prompt for identical copies.** Notepad++ only offers to reload when the new copy differs in date or content.

## Building (Windows, Visual Studio)

Requirements: Visual Studio 2022 or newer with the **Desktop development with C++** workload.

1. Open **x64 Native Tools Command Prompt for VS** from the Start menu. It is a cmd window, not PowerShell.
2. Build the standalone File Manager (all archive formats built in, no `7z.dll` needed):

   ```
   cd /d <path-to-this-repo>\CPP\7zip\Bundles\Fm
   nmake
   ```

3. The result is `CPP\7zip\Bundles\Fm\x64\7zFM.exe`. You can run it directly from there, or install it (next section).

To rebuild after a change, run `nmake` again in the same folder (`nmake clean` first for a full rebuild). To build every 7-Zip binary instead, run `nmake` in `CPP\7zip`. See [`DOC/readme.txt`](DOC/readme.txt) for other platforms and options.

## Installing the EXE

This replaces the File Manager of a normal 7-Zip installation, so the Start menu shortcut and "Open archive" in Explorer use the fork. The examples assume the repo is at `C:\src\7zip` and 7-Zip is installed in `C:\Program Files\7-Zip`.

Open **PowerShell as administrator** (right-click → *Run as administrator*) and run:

```powershell
# Close the File Manager if it is running
Stop-Process -Name 7zFM -ErrorAction SilentlyContinue

# First time only: keep a backup of the official File Manager
if (-not (Test-Path "C:\Program Files\7-Zip\7zFM.original.exe")) {
  Copy-Item "C:\Program Files\7-Zip\7zFM.exe" "C:\Program Files\7-Zip\7zFM.original.exe"
}

# Install the fork's build
Copy-Item "C:\src\7zip\CPP\7zip\Bundles\Fm\x64\7zFM.exe" "C:\Program Files\7-Zip\7zFM.exe" -Force
```

To check it worked, open a text file from an archive. It should open from `%TEMP%\7zO_<archive name>\`.

To go back to the official File Manager:

```powershell
Copy-Item "C:\Program Files\7-Zip\7zFM.original.exe" "C:\Program Files\7-Zip\7zFM.exe" -Force
```

**Note:** installing or updating official 7-Zip overwrites `7zFM.exe`. After that, run the install commands again. Also delete `7zFM.original.exe` first, so the backup is the new official version.

## Updating to a new 7-Zip release

`main` mirrors official 7-Zip; the change lives on the `archive-named-temp` branch. When a new 7-Zip version comes out:

1. **Update the source** (PowerShell, in `C:\src\7zip`):

   ```powershell
   git remote add upstream https://github.com/ip7z/7zip.git   # first time only
   git fetch upstream
   git checkout main
   git merge --ff-only upstream/main
   git push origin main
   git checkout archive-named-temp
   git rebase main
   git push --force-with-lease
   ```

   If `git rebase` stops with a conflict in `PanelItemOpen.cpp`, upstream changed the same spot. Run `git rebase --abort`, then re-apply the `// Fork change` blocks by hand on top of the new `main`.

2. **Rebuild** in the **x64 Native Tools Command Prompt for VS**:

   ```
   cd /d C:\src\7zip\CPP\7zip\Bundles\Fm
   nmake clean
   nmake
   ```

3. **Reinstall** with the commands in [Installing the EXE](#installing-the-exe).

4. **Optional: share the build** as a GitHub Release. On the fork's GitHub page, go to **Releases** → **Draft a new release**. Create a tag such as `26.04-fork1` (the 7-Zip version plus your own counter), attach `CPP\7zip\Bundles\Fm\x64\7zFM.exe`, and publish. Don't commit the EXE to the repository itself; the build folder is meant to stay out of git.

## License

Same as 7-Zip: GNU LGPL, with the unRAR license restriction for the RAR code and BSD licenses for a few files. See [`DOC/License.txt`](DOC/License.txt). This fork is not affiliated with or endorsed by Igor Pavlov or the official 7-Zip project.
