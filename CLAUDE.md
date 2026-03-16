# CLAUDE.md — Claude Code Remotion Guide

## 0. Top Priority Principles

- **Project type**: Documentation repository — technical guide for Remotion x Claude Code programmatic video production
- **Primary language/stack**: Markdown, SVG, HTML
- **Language rules**: All documents, code examples, commit messages, and PR descriptions in English. Korean translation exists as a separate file (`korean.md`)
- **Absolute constraints**: Never modify `LICENSE`. Never overwrite guide content without explicit user instruction

## 1. Repository Structure

```
claude-code-remotion-guide/
├── README.md                 # Project overview and quick start
├── CLAUDE.md                 # This file — Claude Code project rules
├── LICENSE                   # CC BY 4.0
├── CODE_OF_CONDUCT.md        # Contributor Covenant v2.1
├── CONTRIBUTING.md           # Contribution guidelines
├── SECURITY.md               # Security policy
├── .gitignore                # Git ignore rules
├── .gitattributes            # Line ending and diff driver config
├── english.md                # Full technical guide (English)
├── korean.md                 # Full technical guide (Korean)
├── repo-card.html            # GitHub social preview card
├── assets/                   # SVG diagrams referenced in guides
│   ├── 01-architecture-overview.svg
│   ├── 02-animation-cheatsheet.svg
│   └── 03-workflow-pipeline.svg
└── .github/
    ├── PULL_REQUEST_TEMPLATE.md
    └── ISSUE_TEMPLATE/
        ├── bug-report.yml
        └── feature-request.yml
```

## 2. Development Rules

### File Naming

- Markdown files: kebab-case (`feature-name.md`)
- SVG assets: number-prefixed kebab-case (`01-topic-name.svg`)

### Document Style

- One H1 (`#`) per file
- Blockquote summary immediately after the title
- Code blocks with language tags (` ```bash `, ` ```typescript `, etc.)
- Copy-paste ready examples

### Quality Checks

- All internal links must resolve
- Code examples must be syntactically valid
- Cross-references between English and Korean guides must stay consistent

## 3. Git Rules

### Allowed Commands

```
git status
git diff [--cached] [--stat] [file]
git log [--oneline] [-n N]
git add <specific-files>
git commit -m "..."
git push                     # Only when explicitly requested
```

### Prohibited Actions

```
git add -A
git add .
git push --force
git reset --hard
```

### Commit Message Format

```
docs(<scope>): <subject>
```

**Scopes**: `guide`, `assets`, `claude`, `repo`

**Examples**:
```bash
git commit -m "docs(guide): add cloud rendering section"
git commit -m "docs(assets): update architecture diagram"
git commit -m "docs(repo): add GitHub issue templates"
```

### Branch Naming

| Prefix | Purpose |
|--------|---------|
| `feature/` | New documents or sections |
| `fix/` | Corrections, typo fixes |
| `chore/` | Repo config, CI, metadata |

## 4. Workflow

### Plan Mode

- Enter Plan Mode before any multi-file change
- Re-plan if scope changes during implementation

### Subagent Strategy

- Delegate research tasks to subagents
- Keep main context focused on the current editing task

### Verification Before Completion

- Confirm all internal links resolve
- Check cross-reference consistency between English and Korean guides
- Validate YAML templates with proper syntax

### Autonomous Problem Solving

- Fix issues directly — do not ask for step-by-step permission on obvious corrections
- Flag ambiguous decisions for user review

## 5. Task Management

- Plan tasks in `tasks/todo.md` with checkable items
- Confirm plan with user before starting
- Track progress by checking off completed items
- Record lessons learned in `tasks/lessons.md`

## 6. Core Principles

1. **Simplicity first** — prefer clear, direct documentation over clever formatting
2. **Fix root causes** — correct the source of errors, not symptoms
3. **Minimal impact** — make the smallest change that achieves the goal
