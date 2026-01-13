# What's New: Auto-Save Feature

## Quick Summary

✅ **All reviews are now automatically saved as markdown files**

### Before
- Review shown in conversation only
- No permanent record
- Difficult to reference later

### After
- Review saved to `.review/uncommitted-TIMESTAMP-BRANCH.md`
- Full audit trail
- Easy to share and reference

## How It Works

```bash
# User runs
@uncommitted-review

# Agent executes review
[Processes changes...]
[Invokes subagents...]
[Generates review...]

# Agent saves review
Writing to: .review/uncommitted-2025-01-13-143022-main.md
✅ Review saved

# Agent shows in conversation
## Summary
[Review content...]

---
**Review File**: .review/uncommitted-2025-01-13-143022-main.md
```

## Example Output

```markdown
## Summary
Modified 3 files in authentication module adding JWT validation.

## Critical Issues
[Issues...]

## Overall Assessment
- Quality Score: 7/10
- Security Score: 8/10 (from @security-review)
- Performance Score: 6/10 (from @performance-review)
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

You can view, share, or reference this file later.
```

## File Structure

```
project/
├── .review/
│   ├── uncommitted-2025-01-13-143022-main.md
│   ├── uncommitted-2025-01-13-150115-develop.md
│   └── uncommitted-2025-01-13-161245-feature-auth.md
├── src/
└── tests/
```

## Key Benefits

1. **Permanent Record** - Every review saved
2. **Easy Sharing** - Share markdown files
3. **Audit Trail** - Track quality over time
4. **Team Ready** - Commit to git
5. **CI/CD Ready** - Process with automation
6. **Full Metadata** - Branch, timestamp, scores

## Using Saved Reviews

### View Latest Review
```bash
cat $(ls -t .review/*.md | head -1)
```

### List All Reviews
```bash
ls -la .review/
```

### Find Reviews by Branch
```bash
ls .review/ | grep "main"
```

### Commit to Git (Team Sharing)
```bash
git add .review/
git commit -m "Add code reviews"
git push
```

### Gitignore (Personal)
```bash
echo ".review/" >> .gitignore
```

## Troubleshooting

### File Not Created?
1. Restart OpenCode
2. Check permissions in agent frontmatter
3. Manually create directory: `mkdir .review/`

### Can't Write File?
1. Check disk space: `df -h`
2. Check file permissions: `ls -la`
3. Try saving to project root instead

### Wrong Filename?
1. Test commands:
   ```bash
   date +"%Y-%m-%d-%H%M%S"
   git branch --show-current
   ```
2. Check allowed patterns in agent permissions

## Quick Start

```bash
# Install
cp opencode-agent-*.md ~/.config/opencode/agent/

# Restart
opencode

# Use (auto-saves review)
@uncommitted-review

# Check saved file
cat .review/uncommitted-*.md
```

## Documentation

- **FILE_SAVING_GUIDE.md** - Complete documentation
- **AUTO_SAVE_UPDATE.md** - Feature details
- **CHANGES_SUMMARY.txt** - All features summary
- **README.md** - Full project docs

---

**Ready to use!** All reviews will be automatically saved as markdown files.
