---
name: update-docs
description: Update Roadmap and CLAUDE.md after a milestone tag, with verification gate
allowed-tools: Read, Write, Edit, Bash(git *), Bash(wc *), Bash(find *), Grep, Glob
user-invocable: true
---

# Update Documentation After Milestone

You are updating the project's documentation files after a milestone has been tagged.

## Current State (injected)

Latest tag: !`git describe --tags --abbrev=0 2>/dev/null || echo "no tags"`
Current branch: !`git branch --show-current`

Recent commits since last tag (if any):
!`git log --oneline -10`

### File sizes
Main CLAUDE.md: !`wc -l CLAUDE.md` lines

### Current headers
!`head -7 CLAUDE.md`

## Arguments

$ARGUMENTS

If no arguments provided, infer the milestone from the latest tag and recent commits.

## Update Procedure

### Step 1: Determine what changed
Read the recent commits and identify:
- What feature/fix was completed
- Which files were modified (check git diff if needed)
- What the tag name represents

### Step 2: Discover doc files

```bash
# Roadmap
find . -iname "*ROADMAP*.md" -not -path "./.git/*" -not -path "*/node_modules/*" 2>/dev/null

# Changelog
find . -name "CLAUDE_CHANGELOG.md" -not -path "./.git/*" 2>/dev/null

# File versions (when project has source files with version headers)
find . -name "*FILE_VERSIONS*" -not -path "./.git/*" 2>/dev/null
```

### Step 3: Update Roadmap (if found)
- Bump the header version (MINOR for features, PATCH for fixes)
- Add "Changes from vX.Y.Z" section at top (if that pattern exists)
- Add version history entry (if a history table exists)
- Update any phase/task status rows that changed

### Step 4: Update CLAUDE.md
- Bump title version to match Version field
- Bump Version field
- Update Updated date to today
- Update Tag to latest git tag
- Update Status line if needed
- Update Version History table: keep last 5 entries
- If > 5 entries, overflow older ones to `docs/CLAUDE_CHANGELOG.md`

### Step 5: Update satellite files (if they exist)

**Changelog** (`docs/CLAUDE_CHANGELOG.md`):
- Add new version entry at top of table

**File Versions** (`docs/CLAUDE_FILE_VERSIONS.md`):
- Update any file versions that changed (check `git diff` for version header changes)
- Only update this file if it exists — don't create it until the project has source files with version headers

**Additional CLAUDE.md files** (satellite context docs):
- If the project has grown satellite CLAUDE.md files (e.g., `src/.../CLAUDE.md`), update their Version/Date/Tag fields too
- Rotate "Changes from" blocks if applicable: keep latest 2

## Verification Gate (MANDATORY — runs BEFORE commit)

All applicable checks must pass. If ANY fail, report the failure and DO NOT commit.

### V1: Version Consistency
```
- CLAUDE.md title version == Version field
- Tag field matches the latest git tag
- Roadmap header version matches its "Changes from" block (if applicable)
```

### V2: Size Limits
```
- CLAUDE.md < 600 lines
- Any satellite CLAUDE.md < 600 lines
```

### V3: Freshness
```
- "Updated" / "Date" fields are today's date
- "Tag" field references the current tag
```

### V4: Satellite Index
```
- All satellite files referenced in CLAUDE.md "Satellite Files" section exist
```

### V5: Key Document References (optional — runs if Key Documents table found)
```
Scan CLAUDE.md files for markdown tables containing versioned file references (pattern: filename_vX.Y.Z.ext).
For each reference:
1. Extract the referenced path and version from the table row
2. Glob for the actual file: find the latest version matching the base filename pattern
3. Compare: referenced version vs. actual file version on disk
4. FAIL if any referenced file points to a version that doesn't exist OR a newer version exists

Procedure per reference:
  a) Parse path from table: e.g., `docs/guides/VERSION_PROTOCOL_GUIDE_v2.4.0.md`
  b) Extract base name: VERSION_PROTOCOL_GUIDE
  c) Glob: docs/guides/VERSION_PROTOCOL_GUIDE_v*.md
  d) Find highest version file that exists on disk
  e) If referenced version < highest version → STALE, report and auto-fix

For non-versioned filenames with internal versions:
  a) Check if the versioned path exists; if not, look for non-versioned file at alternate locations
  b) Read first 5 lines of the actual file to extract internal version
  c) Report path correction + version update needed

Auto-fix: Update CLAUDE.md table rows to point to correct version. Report each fix.

SKIP if no Key Documents table or versioned file reference tables exist in any CLAUDE.md.
```

Skip checks that don't apply (e.g., V4 if no satellites exist yet, V5 if no Key Documents table).

## Output Format

After all updates, report:
```
Files updated:
  - [list each file and what changed]

Verification:
  V1 Version Consistency: PASS/FAIL (details)
  V2 Size Limits: PASS/FAIL (N lines)
  V3 Freshness: PASS/FAIL
  V4 Satellite Index: PASS/FAIL or SKIP
  V5 Key Document References: PASS/SKIP (N refs checked, N stale, N auto-fixed)

Ready to commit: YES/NO
```

If ready, stage the doc files and commit with message format:
```
docs: update docs for <milestone description>
```

Do NOT push unless explicitly asked.

## Growth Plan

As the project grows, this skill will need updates:

1. **Phase 1+**: Add `docs/CLAUDE_FILE_VERSIONS.md` — track source file header versions
2. **When components grow**: Add satellite CLAUDE.md files per component
3. **When hooks are added**: Add `docs/CLAUDE_HOOKS_REFERENCE.md` tracking
