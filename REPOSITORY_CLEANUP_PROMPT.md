# Repository Cleanup & Organization Prompt

**Purpose:** Comprehensive prompt for cleaning up repository cruft, organizing files, removing outdated artifacts, and establishing clear structure.

**Usage:** Feed this prompt to an AI assistant (Claude, GPT-4, etc.) or use as a checklist for manual cleanup.

---

## Context & Objectives

You are tasked with performing a thorough cleanup and reorganization of a software repository that has accumulated cruft over time. The repository likely contains:

- Temporary files and logs
- Outdated documentation and session notes
- Deprecated feature files and scripts
- Poor organization with too many files at root level
- Inconsistent file placement (docs, tests, configs scattered)
- Generated files that should be gitignored
- Cache directories and build artifacts
- Duplicate or superseded documentation

**Your Goals:**
1. Identify all cruft and temporary files
2. Organize files into logical directory structure
3. Archive or remove outdated content
4. Consolidate documentation
5. Establish clear conventions for file placement
6. Clean up root directory (target: <20 files)
7. Update .gitignore appropriately
8. Create organization guidelines for future

---

## Phase 1: Discovery & Assessment

### 1.1 Inventory Current State

**Root Directory Analysis:**
```bash
# Count items in root
ls -1 | wc -l

# List all files at root (not directories)
find . -maxdepth 1 -type f | sort

# List all directories at root
find . -maxdepth 1 -type d | sort
```

**Document your findings:**
- [ ] Total items in root directory: ___
- [ ] Target: Reduce to <20 essential items
- [ ] Files that clearly don't belong at root: ___
- [ ] Directories that could be consolidated: ___

### 1.2 Identify Git Submodules First

**IMPORTANT:** Before categorizing directories, check for git submodules - these are already separate repositories and should NOT be extracted.

```bash
# Check if repository uses submodules
if [ -f .gitmodules ]; then
    echo "Repository uses git submodules:"
    git submodule status
    cat .gitmodules
else
    echo "No git submodules detected"
fi
```

**Understanding Submodules:**
- Git submodules are separate repositories linked to this repo
- They have their own git history, branches, and remotes
- They appear as subdirectories but are externally managed
- Common pattern: External projects used as dependencies
- **DO NOT extract submodules** - they're already external!

**Document your submodules:**
- [ ] List all submodules: ___
- [ ] Verify each is actively maintained externally
- [ ] Check if any should be removed (unused dependencies)
- [ ] Confirm submodules are in .gitignore (content, not directory)

### 1.3 Identify Cruft Categories

Create an inventory of files by category:

**A. Temporary/Session Files**
- Pattern: `*_COMPLETE.md`, `*_SUMMARY.md`, `*_STATUS.md` with dates
- Pattern: `SESSION_SUMMARY*.md`
- Pattern: `*.log`, `*.cache`
- Pattern: Session-specific files with timestamps
- Examples to look for:
  - `SESSION_SUMMARY_OCT16.md`
  - `.{project}_session` or `.*_session`
  - `{service}_bot.log`
  - `{feature}_poller.log`

**B. Outdated/Superseded Documentation**
- Multiple versions of the same doc (v1, v2, FINAL, COMPLETE)
- Phase completion markers that are now historical
- Migration status files after migration complete
- Implementation summaries after feature shipped
- Examples:
  - `PHASE4_COMPLETE.md`, `PHASE4_COMPLETE_FINAL.md`
  - `IMPLEMENTATION_SUMMARY.md`
  - `BASELINE_EVAL_SUMMARY.md` vs `EVAL_COMPARISON_REPORT.md`

**C. Generated/Build Artifacts**
- Cache directories (`.pytest_cache`, `__pycache__`, `.mypy_cache`)
- Build outputs
- Downloaded files
- Generated images with metadata
- Evaluation result files (JSON, txt)

**D. Test/Demo Files at Root**
- Pattern: `test_*.py` at root level
- Pattern: `demo_*.py` at root level
- Quick test scripts
- Smoke tests outside tests/ directory

**E. Configuration Scattered**
- Configs at root vs configs/ directory
- Multiple .env files
- JSON configs not in configs/

**F. Documentation at Root**
- Should be in docs/ or top-level overview only
- Feature-specific docs
- Tutorial/guide docs
- Architecture/design docs
- Fix/implementation docs

**G. Subprojects/Modules (NOT Submodules)**
- Self-contained directories with own README, requirements.txt
- Could be separate repositories
- Have their own test suites
- **CHECK FIRST:** Is this already a git submodule? If yes, skip extraction
- Examples: standalone tools, UI implementations, pipelines
- Note: If directory is in .gitmodules, it's already external - leave it alone!

---

## Phase 2: Categorize Every File

### 2.1 Root Directory Files - Decision Tree

For each file at root, ask:

**Essential Root Files (KEEP AT ROOT):**
- [ ] `README.md` - Primary project overview
- [ ] `LICENSE` or `LICENSE.txt` - Legal requirement
- [ ] `.gitignore` - Git configuration
- [ ] `.env.example` - Template (never .env itself!)
- [ ] `requirements.txt` or `pyproject.toml` - Dependency management
- [ ] `setup.py` - If installable package
- [ ] `Makefile` or `justfile` - Build commands
- [ ] `CHANGELOG.md` - Version history
- [ ] `CONTRIBUTING.md` - Contributor guidelines
- [ ] Main executable scripts (e.g., `{project}`, `main.sh`, `run.sh`)

**Move to docs/ (DOCUMENTATION):**
```
Criteria: Explains features, architecture, guides, tutorials

Archive if superseded: PHASE*_COMPLETE.md after phase done
Archive if outdated: Implementation docs after feature shipped
Archive if session notes: Daily summaries, work logs

Suggested structure:
docs/
├── architecture/
│   └── ARCHITECTURE.md
├── guides/
│   ├── QUICKSTART.md
│   ├── SETUP.md
│   └── tutorials/
├── design/
│   ├── ADRs/  (Architecture Decision Records)
│   └── proposals/
├── development/
│   ├── TECH_DEBT.md
│   └── REFACTORING_OPTIONS.md
├── features/
│   ├── telegram/
│   ├── filesystem/
│   └── extensions/
└── archive/
    ├── sessions/  (Session summaries)
    ├── phases/    (Phase completion docs)
    └── deprecated/
```

**Move to tests/ (TESTING):**
```
Criteria: test_*.py, demo_*.py, smoke tests, eval scripts

tests/
├── unit/
├── integration/
├── e2e/
├── demos/
├── smoke/
└── fixtures/
```

**Move to configs/ (CONFIGURATION):**
```
Criteria: *.json config files, *.yaml/*.yml configs, config templates, example configs

configs/
├── examples/
├── templates/
├── development/
├── production/
└── README.md  (explain each config)
```

**Move to scripts/ (UTILITIES):**
```
Criteria: Helper scripts, migration scripts, setup scripts, deployment scripts

scripts/
├── setup/
├── migration/
├── maintenance/
├── deployment/
└── utils/
```

**Archive or Delete (CRUFT):**
```
Criteria:
- Temporary session files → DELETE or archive to docs/archive/sessions/
- Log files → DELETE (should be in .gitignore)
- Old evaluation results → archive to results/archive/ or DELETE
- Superseded docs → archive to docs/archive/deprecated/
- Generated images → Keep examples, delete bulk in .gitignore
- Cache directories → DELETE, ensure in .gitignore
```

### 2.2 Subdirectory Organization

**Evaluate each top-level directory:**

For each directory, determine:
- [ ] Is this a core feature? (KEEP)
- [ ] Is this a subproject? (EXTRACT to separate repo)
- [ ] Is this a tool/utility? (MOVE to tools/ or scripts/)
- [ ] Is this documentation? (MERGE into docs/)
- [ ] Is this temporary? (ARCHIVE or DELETE)

**Common subproject indicators:**
- Has own README.md
- Has own requirements.txt or pyproject.toml
- Has own test suite
- Could function independently
- Has own CI/CD or build process
- **Action:** Consider extracting to separate repository

---

## Phase 3: Cleanup Actions

### 3.1 Immediate Deletions (Low Risk)

**Delete these file types:**
```bash
# Cache directories
rm -rf .pytest_cache .mypy_cache __pycache__ .ruff_cache

# Log files (check they're not needed first!)
rm *.log

# OS-specific files
rm .DS_Store
find . -name "._*" -delete  # macOS resource forks

# Editor temp files
rm *~ *.swp *.swo

# Build artifacts (if rebuilding is easy)
rm -rf build/ dist/ *.egg-info/

# Temporary session files (if not needed)
rm .{project}_session
rm .*_session
rm .*_cache
```

### 3.2 Move Documentation

**Create organized docs structure:**
```bash
mkdir -p docs/{architecture,guides,design,development,features,archive}
mkdir -p docs/archive/{sessions,phases,deprecated}
```

**Move files:**
```bash
# Architecture docs
mv ARCHITECTURE.md docs/architecture/
mv REFACTORING_*.md docs/development/

# Guides
mv QUICKSTART.md docs/guides/
mv *_GUIDE.md docs/guides/

# Phase documentation (if historical)
mv PHASE*.md docs/archive/phases/

# Session summaries
mv SESSION_SUMMARY*.md docs/archive/sessions/

# Implementation docs (post-implementation)
mv *_IMPLEMENTATION_COMPLETE.md docs/archive/deprecated/
mv *_SUMMARY.md docs/archive/sessions/

# Feature documentation
mv {FEATURE}_*.md docs/features/{feature}/
mv {INTEGRATION}_*.md docs/features/{integration}/
mv EXTENSIONS.md docs/features/
```

### 3.3 Move Tests

```bash
# Move root-level test files
mv test_*.py tests/
mv demo_*.py tests/demos/
mv smoke_test*.py tests/smoke/
```

### 3.4 Consolidate Configuration

```bash
# Move scattered configs
mv {feature}_config.json configs/
mv {module}_*.json configs/
mv *_config.json configs/ (if not already there)
mv *_config.yaml configs/

# Keep .env.example at root, move .env to .gitignore
```

### 3.5 Archive Results/Outputs

```bash
mkdir -p results/archive/
mv *_results.txt results/archive/
mv *_results.json results/archive/
mv baseline_*.txt results/archive/
```

---

## Phase 4: Gitignore Update

### 4.1 Review Current .gitignore

**Ensure these are ignored:**
```gitignore
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
*.egg-info/
.installed.cfg
*.egg

# Virtual environments
.env
.venv
env/
venv/
ENV/

# IDEs
.vscode/
.idea/
*.swp
*.swo
*~
.DS_Store

# Testing
.pytest_cache/
.coverage
htmlcov/
.tox/
.mypy_cache/
.ruff_cache/

# Logs
*.log
logs/
*.log.*

# Temporary files
*.tmp
*.temp
.cache/
.temp/
*_cache/

# Session files
.{project}_session
.*_session
.session_*

# Generated files
generated/
outputs/
downloads/

# Secrets
.env
.env.local
*.key
*.pem
*_token.json
credentials.json

# OS
.DS_Store
Thumbs.db
._*

# Project-specific temporary files
SESSION_SUMMARY_*.md
*_COMPLETE_*.md
*_STATUS_*.md
```

---

## Phase 5: Establish Organization Guidelines

### 5.1 Create ORGANIZATION.md

Document file placement rules:

```markdown
# Repository Organization Guidelines

## Directory Structure

### Root Level (Maximum 20 files)
ALLOWED:
- README.md, LICENSE, CHANGELOG.md, CONTRIBUTING.md
- .gitignore, .env.example
- requirements.txt, pyproject.toml, setup.py
- Makefile, justfile
- Main executable scripts ({project}, main.sh, run.sh)

NOT ALLOWED:
- Test files (use tests/)
- Documentation (use docs/)
- Configuration (use configs/)
- Temporary files
- Log files

### Core Directories

**src/** or **{project_name}/**
- Production code only
- Organized by module/feature
- No test files
- No documentation

**tests/**
- All test files
- Mirrors src/ structure
- Subdirectories: unit/, integration/, e2e/, demos/

**docs/**
- All documentation
- Subdirectories by category: architecture/, guides/, features/
- Archive old docs to docs/archive/

**configs/**
- Configuration files only
- Examples in configs/examples/
- Templates in configs/templates/

**scripts/**
- Utility scripts
- Setup/installation scripts
- Migration scripts
- Maintenance scripts

**tools/**
- Standalone tools and utilities
- Each tool in its own subdirectory

### File Naming Conventions

**Documentation:**
- Descriptive names: `ARCHITECTURE.md`, not `arch.md`
- No dates in filenames (use git history)
- No status markers in names (COMPLETE, FINAL, v2)

**Tests:**
- Prefix with `test_`: `test_feature.py`
- Demos: `demo_feature.py`
- Fixtures: `conftest.py`, `fixtures.py`

**Configuration:**
- Descriptive: `agent_config.json`, not `config.json`
- Environment-specific: `development.json`, `production.json`

**Scripts:**
- Action-oriented: `setup.sh`, `migrate_db.py`
- Organized by purpose in subdirectories

### What Goes Where?

**When creating a new file, ask:**

1. Is this production code?
   → src/{module}/

2. Is this a test?
   → tests/{category}/

3. Is this documentation?
   → docs/{category}/

4. Is this configuration?
   → configs/{environment}/

5. Is this a utility script?
   → scripts/{purpose}/

6. Is this temporary?
   → Don't commit it! Add to .gitignore

### Archive Policy

**Move to archive when:**
- Documentation is superseded (keep latest only)
- Feature is deprecated
- Migration is complete (migration docs)
- Phase is finished (phase docs)
- Session/daily notes are >1 month old

**Archive locations:**
- docs/archive/deprecated/ - Old feature docs
- docs/archive/phases/ - Completed phase docs
- docs/archive/sessions/ - Session summaries
- results/archive/ - Old evaluation results

**Delete entirely when:**
- Temporary session files >1 week old
- Log files (should be gitignored)
- Build artifacts (can be regenerated)
- Cache directories
```

### 5.2 Create .githooks/ (Optional)

Pre-commit hook to prevent root directory bloat:

```bash
#!/bin/bash
# .githooks/pre-commit

# Count files in root (excluding directories and allowed files)
allowed_root_files=(
    "README.md" "LICENSE" "LICENSE.txt" ".gitignore"
    ".env.example" "requirements.txt" "pyproject.toml"
    "setup.py" "Makefile" "CHANGELOG.md" "CONTRIBUTING.md"
    "{project}" "main.sh" "run.sh"
)

root_files=$(git diff --cached --name-only --diff-filter=A | grep "^[^/]*$")

for file in $root_files; do
    if [[ ! " ${allowed_root_files[@]} " =~ " ${file} " ]]; then
        echo "ERROR: File '$file' added to root directory"
        echo "Root directory should contain <20 essential files only"
        echo "Please move to appropriate subdirectory:"
        echo "  - docs/ for documentation"
        echo "  - tests/ for tests"
        echo "  - configs/ for configuration"
        echo "  - scripts/ for utility scripts"
        exit 1
    fi
done
```

---

## Phase 6: Subproject Extraction

### 6.1 Identify Candidate Subprojects (Excluding Submodules)

**CRITICAL FIRST STEP:** Check for git submodules
```bash
# List all submodules
git submodule status

# Check if directory is a submodule
if grep -q "path = dirname" .gitmodules 2>/dev/null; then
    echo "dirname is a submodule - SKIP extraction"
else
    echo "dirname is a regular directory - can be extracted if needed"
fi
```

**Indicators a directory is a candidate for extraction:**
- [ ] Has its own README.md
- [ ] Has its own requirements.txt or pyproject.toml
- [ ] Has its own tests/ directory or test suite
- [ ] Could function as standalone tool/library
- [ ] Has minimal dependencies on parent project
- [ ] Has separate purpose/domain from main project
- [ ] **NOT listed in .gitmodules** (if it is, it's already external!)

**Example scenarios:**

**Already External (Git Submodules) - LEAVE ALONE:**
```bash
# Example from .gitmodules:
[submodule "{library-name}"]
    path = {library-name}
    url = git@github.com:{org}/{library-name}.git
[submodule "{ui-framework}"]
    path = {ui-framework}
    url = git@github.com:{org}/{ui-framework}.git
[submodule "{tool-name}"]
    path = {tool-name}
    url = git@github.com:{org}/{tool-name}.git
```
These are already separate repositories! They're well-organized. Skip extraction.

**Candidates for Extraction (Regular Directories):**
- Large feature modules not yet separated
- Standalone tools without their own repo
- Self-contained utilities that could be shared
- Experimental features that grew too large

### 6.2 Submodule vs Extraction Decision Tree

```
Is the directory in .gitmodules?
├─ YES → It's a git submodule
│         ├─ Is it actively used?
│         │   ├─ YES → Keep as submodule, ensure it's up to date
│         │   └─ NO → Consider removing: git submodule deinit <path>
│         └─ Document in repository README
└─ NO → It's a regular directory
          ├─ Should it be extracted?
          │   ├─ YES → Follow extraction process below
          │   └─ NO → Keep as part of main repo
          └─ Could it become a submodule?
                ├─ YES → Extract to new repo, then add as submodule
                └─ NO → Keep as monorepo component
```

### 6.3 Extraction Process (For Non-Submodule Directories Only)

For each directory that should be extracted (and is NOT already a submodule):

1. **Create new repository**
   ```bash
   mkdir {subproject-name}
   cd {subproject-name}
   git init
   ```

2. **Copy subproject directory**
   ```bash
   # From original repo
   cp -r path/to/subproject/* ../new-repo/
   ```

3. **Create independent setup**
   - Own README.md
   - Own requirements.txt
   - Own .gitignore
   - Own CI/CD configuration
   - Own tests/

4. **Update original repo**
   - Remove subproject directory
   - Add as dependency (if needed): `{subproject-name}>=0.1.0`
   - Update imports
   - Update documentation

5. **Publish subproject**
   - Git repository
   - PyPI package (if library)
   - Documentation site

6. **Optionally add back as submodule** (if still needed as dependency)
   ```bash
   # Remove original directory
   git rm -r path/to/subproject
   git commit -m "Remove {subproject}, now external"

   # Add as submodule
   git submodule add {repo-url} path/to/subproject
   git commit -m "Add {subproject} as submodule"
   ```

### 6.4 Managing Existing Submodules

If your repository already uses submodules effectively (good!), maintain them:

**Best Practices:**
```bash
# Update all submodules to latest
git submodule update --remote --merge

# Check submodule status
git submodule status

# Document in README.md
echo "## Submodules" >> README.md
echo "This repository uses the following external projects as submodules:" >> README.md
git config --file .gitmodules --get-regexp path | while read key value; do
    echo "- \`$value\` - [description]" >> README.md
done
```

**Update .gitignore for submodules:**
```gitignore
# Submodule contents are managed by git submodules
# Don't ignore the submodule directories themselves
# But ignore any build artifacts or local configs within them
*/build/
*/dist/
*/.env
```

**Document submodules in main README:**
```markdown
## External Dependencies (Git Submodules)

This repository includes the following projects as submodules:

- **{submodule1}/** - {Brief description of functionality}
  - Repository: https://github.com/{org}/{submodule1}
  - Used for: {Purpose in this project}

- **{submodule2}/** - {Brief description of functionality}
  - Repository: https://github.com/{org}/{submodule2}
  - Used for: {Purpose in this project}

- **{submodule3}/** - {Brief description of functionality}
  - Repository: https://github.com/{org}/{submodule3}
  - Used for: {Purpose in this project}

To clone with submodules:
\`\`\`bash
git clone --recurse-submodules <repo-url>
\`\`\`

To update submodules:
\`\`\`bash
git submodule update --remote --merge
\`\`\`
```

---

## Phase 7: Documentation Consolidation

### 7.1 Merge Duplicate/Similar Docs

**Look for:**
- Multiple versions: `README.md`, `README_v2.md`, `README_FINAL.md`
- Status variations: `FEATURE.md`, `FEATURE_COMPLETE.md`, `FEATURE_STATUS.md`
- Outdated alternatives: `OLD_ARCHITECTURE.md`, `ARCHITECTURE_NEW.md`

**Process:**
1. Read all versions
2. Identify most current/complete
3. Archive others to docs/archive/deprecated/
4. Add note in archive: "Superseded by docs/{category}/CURRENT.md on YYYY-MM-DD"

### 7.2 Create Documentation Index

**docs/README.md**
```markdown
# Documentation Index

## Getting Started
- [Quick Start](guides/QUICKSTART.md)
- [Installation](guides/INSTALLATION.md)
- [Configuration](guides/CONFIGURATION.md)

## Architecture
- [System Architecture](architecture/ARCHITECTURE.md)
- [Component Design](architecture/COMPONENTS.md)
- [Data Flow](architecture/DATA_FLOW.md)

## Features
- [Feature Index](features/README.md)
- [Extensions System](features/EXTENSIONS.md)
- [Tool System](features/TOOLS.md)

## Development
- [Technical Debt](development/TECH_DEBT.md)
- [Refactoring Plans](development/REFACTORING_OPTIONS.md)
- [Testing Guide](../tests/README.md)

## Archive
- [Deprecated Features](archive/deprecated/)
- [Completed Phases](archive/phases/)
- [Session Notes](archive/sessions/)
```

---

## Phase 8: Create Maintenance Guidelines

### 8.1 Document Cleanup Schedule

**MAINTENANCE.md**
```markdown
# Repository Maintenance Schedule

## Weekly
- [ ] Review and delete log files
- [ ] Clear temporary caches
- [ ] Archive session summaries >1 week old

## Monthly
- [ ] Review docs/archive/, delete >6 months old
- [ ] Update TECH_DEBT.md status
- [ ] Review root directory file count (target: <20)
- [ ] Run security audit (pip-audit, safety)

## Quarterly
- [ ] Review subproject extraction opportunities
- [ ] Consolidate duplicate documentation
- [ ] Update .gitignore patterns
- [ ] Review and merge open cleanup branches

## Before Major Release
- [ ] Archive all phase/migration docs
- [ ] Update CHANGELOG.md
- [ ] Clean all TODO/FIXME comments
- [ ] Remove all test/debug print statements
- [ ] Verify all docs are current
```

---

## Phase 9: Validation

### 9.1 Cleanup Checklist

**Root Directory:**
- [ ] <20 files in root (excluding .git)
- [ ] Only essential files (README, LICENSE, requirements, etc.)
- [ ] No test files
- [ ] No temporary/log files
- [ ] No documentation (except README)

**Documentation:**
- [ ] All docs in docs/ directory
- [ ] Clear categorical organization
- [ ] No duplicates
- [ ] Archive directory for old content
- [ ] Index file (docs/README.md)

**Tests:**
- [ ] All tests in tests/ directory
- [ ] Organized by type (unit, integration, e2e)
- [ ] No tests at root

**Configuration:**
- [ ] All configs in configs/ directory
- [ ] .env.example at root (not .env)
- [ ] Clear organization (examples, templates, environments)

**Gitignore:**
- [ ] All temporary files ignored
- [ ] All logs ignored
- [ ] All secrets ignored
- [ ] All generated files ignored
- [ ] All cache directories ignored

**Organization:**
- [ ] ORGANIZATION.md created
- [ ] Clear guidelines for file placement
- [ ] Archive policy documented
- [ ] Naming conventions established

**Subprojects & Submodules:**
- [ ] Git submodules identified and documented
- [ ] Submodules are up to date
- [ ] Candidate directories for extraction identified
- [ ] Extraction completed for non-submodule directories
- [ ] Remaining directories are core features
- [ ] No duplicated code
- [ ] Submodules documented in README

### 9.2 Size Metrics

**Before Cleanup:**
```bash
echo "Root files: $(find . -maxdepth 1 -type f | wc -l)"
echo "Root size: $(du -sh . | cut -f1)"
echo "Total files: $(find . -type f | wc -l)"
```

**After Cleanup:**
```bash
echo "Root files: $(find . -maxdepth 1 -type f | wc -l)"  # Target: <20
echo "Root size: $(du -sh . | cut -f1)"
echo "Total files: $(find . -type f | wc -l)"
echo "Docs files: $(find docs/ -type f | wc -l)"
echo "Test files: $(find tests/ -type f | wc -l)"
```

**Success Criteria:**
- Root files reduced by 70%+
- All cruft removed or archived
- Clear organization established
- Documentation consolidated
- Tests organized
- Gitignore comprehensive

---

## Phase 10: Future Prevention

### 10.1 Establish Conventions

**Team Guidelines:**
1. Never commit to root unless essential
2. Always categorize new files immediately
3. Archive completed work weekly
4. Delete temporary files before commit
5. Use meaningful names (no dates, no status markers)
6. One source of truth for documentation
7. Keep .gitignore updated

### 10.2 Automation

**Pre-commit hooks:**
- Prevent root directory bloat
- Check for secrets
- Validate file organization
- Run linters
- Check for temporary files

**CI checks:**
- Root directory file count
- Documentation links valid
- No log files committed
- No temporary files committed
- Gitignore coverage

**Scheduled tasks:**
- Weekly cleanup job
- Monthly documentation review
- Quarterly organization audit

---

## Example Execution Plan

### Week 1: Discovery
- [ ] Run discovery scripts
- [ ] **Check for git submodules** (critical first step!)
- [ ] Categorize all files (excluding submodule contents)
- [ ] Create migration plan
- [ ] Document current state
- [ ] Document submodules in README

### Week 2: Quick Wins
- [ ] Delete obvious cruft (logs, cache, .DS_Store)
- [ ] Update .gitignore
- [ ] Move test files to tests/
- [ ] Archive old session summaries

### Week 3: Documentation
- [ ] Create docs/ structure
- [ ] Move all documentation
- [ ] Consolidate duplicates
- [ ] Create index
- [ ] Archive old docs

### Week 4: Deep Organization
- [ ] Move configs to configs/
- [ ] Organize scripts
- [ ] Extract subprojects (non-submodule directories only)
- [ ] Update submodules if needed
- [ ] Document submodule usage
- [ ] Create ORGANIZATION.md
- [ ] Set up pre-commit hooks

### Week 5: Validation
- [ ] Run validation checklist
- [ ] Measure improvements
- [ ] Update team documentation
- [ ] Create maintenance schedule

---

## Success Metrics

**Quantitative:**
- Root directory files: From ___ to <20 (70%+ reduction)
- Total repository size: ___% reduction
- Documentation files consolidated: From ___ locations to 1
- Test organization: 100% in tests/ directory
- Configuration organization: 100% in configs/ directory

**Qualitative:**
- Clear file organization
- Easy to find files
- No ambiguity about where to put new files
- New contributors can navigate easily
- Reduced cognitive load
- Professional appearance

---

## Appendix: Common Pitfalls

**Avoid These Mistakes:**

1. **Deleting too aggressively** - Archive first, delete later
2. **Losing git history** - Use `git mv` not `mv` then `git add`
3. **Breaking imports** - Update all imports after moves
4. **Inconsistent naming** - Establish conventions first
5. **Over-organizing** - Don't create 100 subdirectories
6. **Ignoring dependencies** - Check what depends on moved files
7. **No documentation** - Always update README/docs after changes
8. **One-time cleanup** - Establish ongoing maintenance
9. **No team buy-in** - Get agreement on organization guidelines
10. **Perfectionism paralysis** - Done is better than perfect
11. **Extracting git submodules** - They're already separate repos! Don't try to re-extract them
12. **Modifying submodule contents** - Make changes in the submodule's repo, not here
13. **Forgetting to update submodules** - Run `git submodule update` after pulling
14. **Committing submodule changes in main repo** - Commit them in the submodule repo instead

---

## Templates

### Archive Note Template
```markdown
# [Original Filename]

**Status:** ARCHIVED
**Date:** YYYY-MM-DD
**Reason:** [Superseded/Completed/Deprecated]
**Superseded By:** [Link to current version if applicable]

---

[Original content preserved below]
```

### Documentation Move Commit Message
```
docs: Reorganize documentation structure

- Move [X] files from root to docs/
- Consolidate [feature] documentation
- Archive [Y] outdated documents
- Create docs/ index

Reduces root directory from [X] to [Y] files.
```

---

**End of Cleanup Prompt**

This prompt provides a comprehensive framework for repository cleanup. Adapt the specific categories, thresholds, and organization to fit your project's needs.
