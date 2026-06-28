# Baidu Baike Scholar Entry Drafts

This repository is a Markdown mirror for Baidu Baike scholar/person entry drafts.
It is maintained from the Markdown files currently placed directly under `D:\`.

## Purpose

The repository exists so that the current Baidu Baike draft set can be reviewed,
versioned, and shared through GitHub while keeping `D:\` as the only source of
truth.

The numbered Markdown files in the repository root are mirror files. They should
not be edited in the repository first. Edit or replace the corresponding source
file in `D:\`, then run the sync workflow.

`README.md` is repository documentation and is not part of the mirrored draft
set.

## Source Scope

Only files that match all of these conditions are mirrored:

- The file is directly under `D:\`.
- The file extension is `.md`.
- The file is a regular file.

Files in subdirectories under `D:\` are ignored. Non-Markdown files are ignored.
The sync workflow must never modify, rename, delete, or reformat the source files
in `D:\`.

## Ordering Rule

Before copying, source Markdown files are sorted by:

1. `LastWriteTimeUtc`, newest first.
2. File name, ascending, only when the modification time is exactly identical.

The repository mirror is rebuilt from that ordered list. Each mirrored file is
named with a three-digit sequence prefix:

```text
001_原文件名.md
002_原文件名.md
003_原文件名.md
```

This makes the GitHub file list follow the same order as the `D:\` root
Markdown files sorted by latest modification time.

## Sync Policy

Use `D:\` root Markdown files as the only source.

- If a Markdown file exists in `D:\` root, it must exist in the repository mirror.
- If a mirrored Markdown file no longer corresponds to a `D:\` root file, delete
  it from the repository.
- If the source content changed, update the mirrored file.
- If the source content did not change, avoid unnecessary content changes.
- Do not touch non-Markdown files or unrelated repository configuration.
- Keep `README.md` as documentation outside the mirrored draft set.

## Standard Workflow

Start from a clean repository:

```powershell
$repo = 'C:\Users\12644\Documents\Codex\workspaces\baidu-baike-scholar-entry-drafts'
git -C $repo status --short
git -C $repo pull --ff-only origin main
```

If `git status --short` reports local changes, inspect them first and do not
overwrite user work.

Then rebuild the mirror from `D:\` root Markdown files:

```powershell
$repo = 'C:\Users\12644\Documents\Codex\workspaces\baidu-baike-scholar-entry-drafts'
$sourceRoot = 'D:\'

$sourceFiles = Get-ChildItem -LiteralPath $sourceRoot -File -Filter '*.md' |
  Sort-Object @{ Expression = 'LastWriteTimeUtc'; Descending = $true },
              @{ Expression = 'Name'; Descending = $false }

Get-ChildItem -LiteralPath $repo -File -Filter '*.md' |
  Where-Object { $_.Name -ne 'README.md' } |
  Remove-Item -Force

$i = 1
foreach ($src in $sourceFiles) {
  $targetName = ('{0:D3}_{1}' -f $i, $src.Name)
  Copy-Item -LiteralPath $src.FullName -Destination (Join-Path $repo $targetName) -Force
  $i++
}
```

## Verification

After syncing, verify all three points before committing:

1. The mirrored file count equals the `D:\` root Markdown file count.
2. The mirrored file names exactly match the expected numbered order.
3. Every mirrored file has the same SHA256 hash as its source file.

The comparison should ignore `README.md` because it is documentation, not a
mirrored draft.

## Commit And Push

Commit only when there are real changes:

```powershell
git -C $repo status --short
git -C $repo add -A
git -C $repo commit -m "Sync Baidu Baike scholar entry drafts"
git -C $repo push origin main
```

If `git status --short` is empty after verification, do not create an empty
commit.

## Reporting Checklist

Each sync run should report:

- Whether changes were pushed.
- The commit hash, if a commit was created.
- Current mirrored Markdown count.
- The first and last mirrored file names.
- Whether any temporary files were created, modified, deleted, or cleaned up.

