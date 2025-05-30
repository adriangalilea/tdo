# tdo Specification v1.0

## Overview

tdo manages your entire life as a task tree, with leaves (tasks.md files) distributed across your filesystem. It uses Taskwarrior as a backend for robust task management and provides git-like conflict resolution.

## Core Contract

### 1. Identity Mapping
- Tasks use dynamic short IDs called "dsids" (e.g., `b4`, `k7m`)
- Dsids are immutable and unique within their project scope
- Each task appears in exactly one markdown file
- Inspired by sqids.org for safe character sets

### 2. Tree Path → Folder Mapping
```
Tree Path                    → File System Path
life.garden.plants           → ~/life/garden/plants/tasks.md
projects.tdo                 → ~/repos/tdo/tasks.md
projects.tdo.core            → ~/repos/tdo/core/tasks.md
personal.health.workout      → ~/health/workout/tasks.md
learning.spanish.verbs       → ~/learning/spanish/verbs/tasks.md
```

Note: Paths like `~/repos/tdo/` are git repositories, enabling task collaboration through version control. Others are regular folders for personal task management.

### 3. Markdown File Format
```markdown
---
tdo: 1.0.0
project: life.garden.plants
tags: gardening, outdoor, plants
last_sync: 2024-01-15T10:30:00Z
sync_hash: sha256:abcd1234...
---

# Garden Plant Tasks

## 🟡 In Progress  

- [ ] [n3] Water tomato plants[^1][^2] for #garden due:2024-03-15 start:2024-01-14 @adrian

## 🔒 Blocked

- [ ] [x7] Schedule CPA meeting!! <- b4

## 📝 TODO

- [ ] [m8] Review tax documents! wait:2024-02-01 due:2024-04-15 scheduled:2024-03-01 until:2024-04-30
- [ ] [b4] Buy organic fertilizer for #garden plants!! #next <- supplies:p2

## ✅ Completed

- [x] [j9] Download tax software end:2024-01-14T15:30:00Z

[^1]: @2024-01-14: Got forms from employer A
[^2]: @2024-01-16: Still waiting on employer B
```

### 4. Task Properties Mapping

| Taskwarrior | Markdown | Notes |
|-------------|----------|-------|
| description | Task text | Main content |
| project | Tree path in frontmatter | Determines file location |
| tags | Inline hashtags | Natural flow: "Get forms from #employer" → +TAGGED |
| depends | <- arrow syntax | Inline: "Task <- s1,s2" (computed reverse as blocks) |
| annotations | Footnote references | `Task[^1]` with `[^1]: Note text` |
| due | due:value inline | Natural language → ISO 8601 → +DUE |
| scheduled | scheduled:value inline | Natural language → ISO 8601 → +SCHEDULED |
| wait | wait:value inline | Natural language → ISO 8601 → +WAITING |
| until | until:value inline | Natural language → ISO 8601 → +UNTIL |
| start | start:timestamp | When work actually began → +ACTIVE |
| end | end:timestamp | When work completed |
| priority | ! suffix in task text | No !=none, !=L, !!=M, !!!=H |
| status | Checkbox + section | pending=[ ], completed=[x] |

### 5. Sync State

Centralized in `~/.task/tdo/state.json`:
```json
{
  "version": "1.0",
  "folders": {
    "/Users/alice/projects/website": {
      "last_sync": "2024-01-15T10:30:00Z",
      "sync_id": "sync_20240115_103000",
      "tasks": {
        "a1b2c3d4-e5f6-g7h8-i9j0-k1l2m3n4o5p6": {
          "tw_modified": "2024-01-15T10:00:00Z",
          "md_modified": "2024-01-15T10:15:00Z", 
          "md_file": "collectors/testing/tasks.md",
          "tw_id": 4,
          "content_hash": "sha256:abcd1234...",
          "sync_status": "clean"
        }
      }
    }
  }
}
```

### 6. Conflict Resolution

#### Conflict Detection
A conflict occurs when:
1. Both TW and MD modified since last sync
2. Content hashes differ
3. Same property changed to different values

#### Conflict Markers
```markdown
## ⚠️ SYNC CONFLICTS (1)

<<<<<<< TASKWARRIOR (modified: 2024-01-15T10:30:00Z)
- [ ] [p7] Research local #garden centers!
  - Due: 2024-01-25
======= MARKDOWN (modified: 2024-01-15T10:32:00Z)  
- [ ] [p7] Research #garden centers!! (ask Sarah for #recommendation)
  - Due: 2024-01-30
>>>>>>> 
```

#### Resolution Commands
```bash
# Accept Taskwarrior version
tdo resolve p7 --theirs

# Accept Markdown version  
tdo resolve p7 --ours

# Manual merge (opens editor)
tdo resolve p7 --merge
```

### 7. Change Operations

#### Supported MD → TW Syncs
1. **Status changes**: `[ ]` → `[x]` (complete task)
2. **New tasks**: New list items with proper format
3. **Description edits**: Change task text
4. **Tag changes**: Add/remove inline hashtags
5. **Moving tasks**: Change section (status change)

#### Supported TW → MD Syncs  
1. All Taskwarrior properties
2. Project changes (moves file)
3. Dependency updates
4. Annotations
5. Custom UDAs

### 8. Safety Mechanisms

1. **Dry Run Mode**: Default for destructive operations
2. **Git Integration**: Rely on git history for rollback
3. **Lock File**: System-wide lock in `~/.task/tdo/sync.lock`
4. **Validation**: Schema validation before writing
5. **Rollback**: Use git to revert changes

### 9. File Structure

```
project/
├── tasks.md                 # Root level tasks
├── collectors/
│   ├── tasks.md            # website.design
│   └── testing/
│       └── tasks.md        # website.design.testing
└── portfolio/
    ├── tasks.md            # website.portfolio
    └── v2/
        └── tasks.md        # website.portfolio.v2

~/.task/tdo/                 # Central tdo data
├── state.json              # Global sync state for all repos
├── sync.lock               # System-wide sync lock
└── config.json             # User preferences
```

### 10. Configuration

Project-specific settings in tasks.md frontmatter:
```yaml
---
tdo: 1.0.0
project: website
tags: backend, api  # Applied to ALL tasks in this file
# Optional: Override default sections
# sections:
#   in_progress: "🟡 In Progress"
#   blocked: "🔒 Blocked"
#   todo: "📝 TODO"
#   done: "✅ Completed"
---
```

Note: The sections shown above are the defaults and don't need to be specified unless you want to customize them.

**Project-wide tags**: Tags in frontmatter apply to every task in the file. This is perfect for:
- Department tags (`engineering`, `marketing`)
- Year tags (`2024`, `q1-2024`)
- Project phase tags (`mvp`, `beta`)

Individual tasks can still add their own tags inline, which combine with project tags.

#### Section → Taskwarrior Virtual Tag Mapping

Each markdown section automatically inherits Taskwarrior virtual tags:

| Markdown Section | Virtual Tags Applied | Description |
|-----------------|---------------------|-------------|
| 🟡 In Progress | +PENDING +ACTIVE | Started but not complete |
| 🔒 Blocked | +PENDING +BLOCKED | Has unmet dependencies |
| 📝 TODO | +PENDING | Default for new tasks |
| ✅ Completed | +COMPLETED | Checkbox marked [x] |

Additional computed states:
- Tasks in TODO with no blockers → `+READY`
- Tasks in Blocked with met dependencies → auto-move to TODO
- Tasks with `#waiting` or future start date → `+WAITING`
- Past due tasks in any pending section → `+OVERDUE`

This mapping ensures your markdown sections align perfectly with Taskwarrior's filtering and reporting capabilities.

Global user preferences in `~/.task/tdo/config.json`:
```json
{
  "version": "1.0",
  "sync_mode": "bidirectional",
  "conflict_strategy": "manual",
  "auto_sync": false,
  "exclude_patterns": [
    "**/node_modules/**",
    "**/.git/**"
  ]
}
```

#### Priority System

Tasks use `!` suffixes that map directly to Taskwarrior priorities:
- No `!` = no priority
- `!` = priority:L (Low)
- `!!` = priority:M (Medium)
- `!!!` = priority:H (High)

#### Special Tags

**#next** - Marks tasks for the "next" report with massive urgency boost:
- Adds +15 to urgency (customizable via `urgency.user.tag.next.coefficient`)
- Appears at top of `task next` report
- Use sparingly - only 1-3 tasks should have #next
- Example: `[n7] Review #legal contracts!!! #next`

This lets you override the algorithm and explicitly say "this is what I'm doing next" regardless of other factors.

#### Natural Language Dates

Dates use keyword:value syntax with natural language parsing (inspired by chrono.js). The @ symbol is reserved for people mentions:

```markdown
- [ ] [k4] Plant spring flowers due:april-15 @sarah
- [ ] [m8] Review contract wait:monday @john @mary
- [ ] [p7] Start project scheduled:in-3-weeks @team-lead
- [ ] [t2] Call client due:tomorrow-at-3pm @adrian
- [ ] [r9] Quarterly report due:end-of-quarter
```

Supported date types:
- `due:` - When task must be completed
- `wait:` - Hide until this date (→ +WAITING)
- `scheduled:` - Planned start date (→ +SCHEDULED)
- `until:` - Auto-expire if not done by then (→ +UNTIL)
- `start:` - When work actually began (→ +ACTIVE)
- `end:` - When work completed

Natural language examples:
- Relative: `tomorrow`, `in-2-days`, `next-friday`, `in-3-weeks`
- Specific: `april-15`, `2024-04-15`, `jan-1st`
- Smart: `end-of-month`, `next-quarter`, `monday` (next one)
- With time: `tomorrow-at-3pm`, `friday-5:30pm`

Dates are converted to absolute ISO 8601 during sync and the markdown file is updated:

```markdown
# Before sync (what you type):
- [ ] [k4] Plant spring flowers due:april-15
- [ ] [m8] Call client due:tomorrow-at-3pm
- [ ] [p7] Start project wait:next-monday

# After sync (what gets saved):
- [ ] [k4] Plant spring flowers due:2024-04-15
- [ ] [m8] Call client due:2024-01-30T15:00:00
- [ ] [p7] Start project wait:2024-02-05
```

This ensures dates remain unambiguous - "tomorrow" is converted once, not re-interpreted each sync.

#### Dynamic Short IDs (dsids)

Dsids adapt their length based on project size to minimize typing while avoiding collisions:

- **<100 tasks**: 2 characters (e.g., `b4`, `x7`)
- **<1,000 tasks**: 3 characters (e.g., `k7m`, `n2x`)
- **<10,000 tasks**: 4 characters (e.g., `b4xj`, `m9k2`)
- **≥10,000 tasks**: 5 characters (e.g., `x9k2m`, `j7n3p`)

When referencing tasks:
- **Within same project**: Use bare dsid (`b4`)
- **Cross-project**: Use minimal path disambiguation (`garden:b4` or `plants:b4`)
- **Full reference**: `life.garden.plants:b4` (rarely needed)

#### Dependency Model

Tasks express dependencies inline using the `<-` arrow syntax. The inverse relationship (what this task blocks) is computed automatically by tdo, creating a bidirectional dependency graph internally. This keeps the markdown simple - you only maintain one direction.

```markdown
- [ ] [n3] Gather W2s

- [ ] [k4] Plant spring flowers <- n3
```

Multiple dependencies: `Task <- s1, s2, s3`
Cross-project: `Task <- ../other:k4` or `Task <- @projects.webapp:m2`

tdo knows that n3 blocks k4, even though it's only written once.

#### Cross-Project References & Relative Paths

```markdown
# In life.garden.plants/tasks.md
- [ ] [k4] File tax return <- ../../../health/insurance:m2, n3, b4

# Relative syntax options:
../insurance:m2         # Sibling folder
../../:p7              # Parent's dsid 
./receipts:b4          # Child folder
@projects.tdo:k9    # Absolute from root
```

#### Orphan Handling

When a referenced task is deleted or moved:
```markdown
- [ ] [n7] Review #legal contracts!!! <- legal:b4  # ⚠️ ORPHAN: Task not found
```

During sync, tdo:
1. Validates all cross-references
2. Marks orphans with warnings
3. Suggests new paths for moved tasks
4. Never auto-deletes orphan references (manual cleanup required)

### 11. Command Line Interface

```bash
# Initialize in current directory
tdo init

# Sync all tasks
tdo sync [--dry-run]

# Sync specific project
tdo sync website.design

# Show status
tdo status

# Show conflicts
tdo conflicts

# Resolve conflict
tdo resolve <task_id> [--ours|--theirs|--merge]

# Watch mode
tdo watch

# Rollback last sync
tdo rollback

# Clean orphaned files
tdo clean
```

### 12. Error Handling

1. **Network Errors**: Retry with exponential backoff
2. **Parse Errors**: Quarantine file, notify user
3. **Missing Dependencies**: Create placeholder tasks
4. **Corrupted State**: Rebuild from Taskwarrior + git history
5. **Lock Timeout**: Force unlock with `tdo unlock`

### 13. Performance Considerations

1. **Incremental Sync**: Only changed tasks
2. **Batch Operations**: Group TW commands
3. **Async I/O**: Non-blocking file operations
4. **Cache**: Task metadata in memory
5. **Debounce**: Ignore rapid successive changes

### 14. Migration Path

```bash
# Import existing Taskwarrior data
tdo import --source ~/.task

# Import existing markdown files
tdo import --source ./docs/tasks --format markdown

# Export for backup
tdo export --format json > backup.json
```

### 15. Extensibility

- Plugin system for custom formats
- Webhooks for external integrations
- Custom section mappings
- Additional metadata fields
- Alternative conflict strategies

## Implementation Priority

1. **Phase 1**: Core sync engine (TW → MD)
2. **Phase 2**: Bidirectional sync with manual conflicts
3. **Phase 3**: Watch mode and auto-sync
4. **Phase 4**: Advanced conflict resolution
5. **Phase 5**: Plugin system

## Example Workflow

```bash
# Life workflow
$ task add "Research local nurseries" project:life.garden.plants +urgent
Created task 47.

$ tdo sync
→ Created: ~/life/garden/plants/tasks.md
→ Added: [k4] "Research local CPAs"

$ edit ~/life/garden/plants/tasks.md
# Change: "Research local CPAs" → "Research CPAs (ask Sarah for rec)"

$ tdo sync
⚠️  Conflict detected in k4
Run 'tdo conflicts' for details

$ tdo conflicts
Task k4: Description conflict
  TW: "Research local CPAs"
  MD: "Research CPAs (ask Sarah for rec)"

$ tdo resolve k4 --ours
✓ Resolved: Task k4 using markdown version

$ task k4
Research CPAs (ask Sarah for rec)
```

## FAQ

### Q: Should I commit the `.task/` database or just markdown files to git?

**A: Only commit markdown files.** Here's why:

1. **Avoid UUID conflicts**: Each developer maintains their own central `~/.task/` database. Committing `.task/` would create conflicting UUIDs.

2. **Preserve user preferences**: Urgency scores, custom reports, and personal configurations remain local to each developer.

3. **What you "lose" isn't critical for shared tasks**:
   - Task history/timestamps (git already tracks this)
   - Undo/redo data (use git instead)
   - Recurring task templates (rarely needed in project repos)
   - Hook scripts (personal workflow preferences)

4. **What you gain**:
   - Clean, readable task files in git
   - Each developer's urgency calculations apply to shared tasks
   - No binary database conflicts
   - Simplified collaboration

### Q: How does this work with multiple repositories?

When you clone a repo with task files:
1. `tdo sync` reads the markdown files
2. Tasks are imported to your central `~/.task/` database
3. UUIDs are preserved if tasks already exist, or assigned if new
4. Your personal task configurations (urgency, reports) automatically apply

This creates a "hub and spoke" model where your central Taskwarrior database is the hub, and each repository's markdown files are spokes that sync with it.