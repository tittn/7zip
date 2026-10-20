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

3. The result is `CPP\7zip\Bundles\Fm\x64\7zFM.exe`. Run it directly, or replace `C:\Program Files\7-Zip\7zFM.exe` with it (back up the original first; an official 7-Zip update will overwrite it).

To build every 7-Zip binary instead, run `nmake` in `CPP\7zip`. See [`DOC/readme.txt`](DOC/readme.txt) for other platforms and options.

## Staying up to date with upstream

```
git remote add upstream https://github.com/ip7z/7zip.git   # once
git fetch upstream
git rebase upstream/main
git push --force-with-lease
```

## License

Same as 7-Zip: GNU LGPL, with the unRAR license restriction for the RAR code and BSD licenses for a few files. See [`DOC/License.txt`](DOC/License.txt). This fork is not affiliated with or endorsed by Igor Pavlov or the official 7-Zip project.
