# Auto-Save Update: Review Files

## What Changed

All uncommitted review agents now automatically save reviews as markdown files in project root.

## Key Features

### 1. Main Agent (`opencode-agent-uncommitted-review.md`)

**Updated Permissions**:
```yaml
permission:
  write:
    "*.review.md": allow
    "REVIEW*.md": allow
    ".review/*.md": allow
```

**New Process Step 6**:
- **ALWAYS SAVE** review to markdown file
- Generate timestamped filename
- Get current git branch
- Create `.review/` directory if needed
- Use Write tool to save complete review

**Filename Format**:
- Primary: `.review/uncommitted-YYYY-MM-DD-HHMMSS-{branch}.md`
- Alternative: `REVIEW-{branch}-{timestamp}.md`

**Example**:
```
.review/uncommitted-2025-01-13-143022-main.md
.review/uncommitted-2025-01-13-150115-feature-auth.md
```

### 2. Review Metadata

New section added to every review:
```markdown
**Review Metadata**
- Generated: 2025-01-13-14:30:22
- Git Branch: feature-auth
- Files Changed: 3
- Review File: .review/uncommitted-2025-01-13-143022-feature-auth.md
```

### 3. Automatic File Saving Note

Every review includes:
```markdown
## Automatic File Saving

**IMPORTANT**: This review has been automatically saved to:
```
.review/uncommitted-2025-01-13-143022-feature-auth.md
```
```

## Benefits

### 1. **Permanent Record**
- Every review saved as file
- Full history of all reviews
- Easy to reference later

### 2. **Team Collaboration**
- Share review files with team
- Commit reviews to git
- Track code quality over time

### 3. **Audit Trail**
- See what issues were found
- Track improvements over time
- Reference specific decisions

### 4. **Integration Ready**
- Pre-commit hooks can read reviews
- CI/CD can archive reviews
- Git commits can reference reviews

## File Structure

```
project-root/
├── .review/                          # Review directory (auto-created)
│   ├── uncommitted-2025-01-13-143022-main.md
│   ├── uncommitted-2025-01-13-150115-develop.md
│   └── uncommitted-2025-01-13-161245-feature-x.md
├── src/                              # Your code
├── tests/                            # Your tests
└── ...
```

## How It Works

### User Runs Agent

```bash
@uncommitted-review
```

### Agent Executes

```
1. Check git diff --stat
2. Analyze changes
3. Invoke subagents if needed
4. Integrate findings
5. Generate complete review
6. Get timestamp: date +"%Y-%m-%d-%H%M%S"
7. Get branch: git branch --show-current
8. Create filename
9. Save review to file
10. Inform user of file location
```

### Result

```markdown
## Summary
[Review content...]

## Overall Assessment
- Quality Score: 7/10
- Ready to commit: No

---
**Review Metadata**
- Generated: 2025-01-13-14:30:22
- Git Branch: feature-auth
- Files Changed: 3
- Review File: .review/uncommitted-2025-01-13-143022-feature-auth.md

**Note**: This review was enhanced by specialized subagents:
- @security-review: invoked
- @performance-review: invoked

---

## Automatic File Saving

**IMPORTANT**: This review has been automatically saved to:
```
.review/uncommitted-2025-01-13-143022-feature-auth.md
```
```

## Viewing Reviews

### List All Reviews

```bash
ls -la .review/
```

### Read Latest Review

```bash
# Show most recent
cat "$(ls -t .review/*.md | head -1)"

# Or specify file
cat .review/uncommitted-2025-01-13-143022-main.md
```

### Find Reviews by Branch

```bash
ls .review/ | grep "main"
```

### Find Reviews by Date

```bash
ls .review/ | grep "2025-01-13"
```

## Git Integration

### Option 1: Commit Reviews (Team Workflow)

```bash
# Add review directory
git add .review/

# Commit with review reference
git commit -m "Add review for feature X - see .review/uncommitted-*.md"

# Push to share with team
git push
```

**Benefits**:
- Team sees all reviews
- Reviews in commit history
- PR discussion can reference reviews

### Option 2: Gitignore Reviews (Personal Workflow)

```bash
# Add to .gitignore
echo ".review/" >> .gitignore
echo "REVIEW*.md" >> .gitignore

# Commit ignore file
git add .gitignore
git commit -m "Ignore review files"
```

**Benefits**:
- Clean git history
- Personal review history only
- No noise in PRs

## Automated Workflows

### Pre-commit Hook

Create `.git/hooks/pre-commit`:

```bash
#!/bin/bash

echo "🔍 Running code review..."

# Run review (it will auto-save)
opencode run "@uncommitted-review"

# Check review file for critical issues
LATEST_REVIEW=$(ls -t .review/*.md 2>/dev/null | head -1)

if [ -n "$LATEST_REVIEW" ]; then
    if grep -q "## Critical Issues" "$LATEST_REVIEW"; then
        echo "❌ Critical issues found - see $LATEST_REVIEW"
        exit 1
    fi
fi

echo "✅ Review passed"
exit 0
```

Make executable:
```bash
chmod +x .git/hooks/pre-commit
```

### CI/CD Integration

```yaml
# GitHub Actions
- name: Run code review
  run: |
    opencode run "@uncommitted-review"
    
- name: Upload reviews
  uses: actions/upload-artifact@v4
  with:
    name: code-reviews
    path: .review/*.md
    
- name: Check for blocking issues
  run: |
    LATEST_REVIEW=$(ls -t .review/*.md | head -1)
    if grep -q "## Critical Issues" "$LATEST_REVIEW"; then
      echo "Critical issues detected - failing build"
      cat "$LATEST_REVIEW"
      exit 1
    fi
```

## Managing Review History

### Archive Old Reviews

```bash
# Create archive
mkdir -p .review/archive

# Move reviews older than 30 days
find .review/*.md -mtime +30 -exec mv {} .review/archive/ \;
```

### Clean All Reviews

```bash
# ⚠️ WARNING: Deletes all reviews
rm -rf .review/
```

### Compress Old Reviews

```bash
# Create archive
tar -czf reviews-archive-$(date +%Y%m%d).tar.gz .review/*.md

# Remove old uncompressed files
rm .review/*.md
```

## Comparison: Before vs After

| Aspect | Before | After |
|---------|--------|-------|
| Output | Conversation only | Conversation + saved file |
| History | Not preserved | Full file history |
| Sharing | Copy/paste text | Share markdown file |
| CI/CD | Difficult to parse | Easy to process |
| Team access | No | Commit to git |
| Reference | Search conversation | Open file |

## Troubleshooting

### File Not Created

**Symptoms**: Review completes but no `.review/` directory

**Causes**:
1. Agent not loaded (restart OpenCode)
2. Write permissions not active
3. Directory creation failed

**Solutions**:
1. Restart OpenCode completely
2. Verify agent frontmatter has write permissions
3. Check file system permissions
4. Manually create directory: `mkdir .review/`

### Wrong Filename

**Symptoms**: File saved with unexpected name

**Causes**:
1. Date command fails
2. Branch name capture fails

**Solutions**:
1. Test commands:
   ```bash
   date +"%Y-%m-%d-%H%M%S"
   git branch --show-current
   ```
2. Use fallback filename: `REVIEW-latest.md`

### Can't Write File

**Symptoms**: Permission denied error

**Causes**:
1. Pattern doesn't match
2. Disk full
3. File system read-only

**Solutions**:
1. Check pattern matches filename
2. Check disk space: `df -h`
3. Check file system: `mount`

## Best Practices

### 1. Keep Reviews Organized
- Use `.review/` directory
- Consistent naming convention
- Archive old reviews

### 2. Share With Team
- Commit reviews to git
- Reference reviews in PRs
- Discuss review findings in team meetings

### 3. Use Reviews Effectively
- Address critical issues immediately
- Track improvement over time
- Learn from review patterns

### 4. Integrate Into Workflow
- Pre-commit hooks for automatic reviews
- CI/CD for project reviews
- Git hooks for branch-specific rules

### 5. Maintain Review History
- Don't delete old reviews
- Archive by month/quarter
- Backup reviews to cloud storage

## Future Enhancements

Potential improvements:
1. **Review Dashboard**: Visual summary of all reviews
2. **Trend Analysis**: Track code quality over time
3. **Issue Categories**: Group similar issues
4. **Review Comparison**: Side-by-side comparison
5. **Auto-assign**: Assign issues to team members
6. **Review Templates**: Different review types
7. **Review Comments**: Add comments to saved reviews

## Summary

✅ **Auto-save enabled** - All reviews saved as markdown
✅ **Timestamped files** - Easy to track history
✅ **Organized directory** - `.review/` for all reviews
✅ **Full metadata** - Branch, timestamp, scores included
✅ **Git ready** - Easy to commit or gitignore
✅ **CI/CD compatible** - Can be processed by automation
✅ **Team friendly** - Share reviews easily
✅ **Audit trail** - Permanent record of all reviews

### Installation

```bash
# Copy updated agents
cp opencode-agent-*.md ~/.config/opencode/agent/

# Restart OpenCode
opencode

# Use - review will auto-save
@uncommitted-review
```

### Files Updated

1. `opencode-agent-uncommitted-review.md` - Auto-save enabled
2. `opencode-agent-security-review.md` - Hidden (unchanged)
3. `opencode-agent-performance-review.md` - Hidden (unchanged)
4. `FILE_SAVING_GUIDE.md` - Complete documentation

### Testing

```bash
# Make a small change
echo "// test" >> app.js

# Run review
@uncommitted-review

# Check if file was saved
ls -la .review/

# View saved review
cat .review/uncommitted-*.md
```

### Documentation

- **FILE_SAVING_GUIDE.md** - Complete file saving documentation
- **AGENT_UPDATE_SUMMARY.md** - Delegation system details
- **agent-flow.md** - Visual flow diagrams
- **README.md** - Full project documentation
- **QUICK_REFERENCE.md** - Quick reference guide

### Support

For detailed documentation:
- See `FILE_SAVING_GUIDE.md` for complete file saving implementation
- See `AGENT_UPDATE_SUMMARY.md` for delegation system details
- See `README.md` for full project documentation

---

**Updated: 2025-01-13**
**Feature: Auto-save reviews as markdown files**
**Status: ✅ Complete and tested**
