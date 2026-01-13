# File Saving Implementation Guide

## How Agents Save Reviews

The uncommitted review agents are configured to automatically save reviews as markdown files in the project root.

## File Structure

```
project-root/
├── .review/
│   ├── uncommitted-2025-01-13-143022-feature-auth.md
│   ├── uncommitted-2025-01-13-150115-bug-fix.md
│   └── uncommitted-2025-01-13-161245-refactor.md
├── REVIEW-feature-auth-20250113143022.md (alternative format)
├── REVIEW-bug-fix-20250113150115.md (alternative format)
└── ...
```

## Permissions Configuration

### Main Agent (`opencode-agent-uncommitted-review.md`)

```yaml
permission:
  write:
    "*.review.md": allow
    "REVIEW*.md": allow
    ".review/*.md": allow
  edit: deny
  bash:
    "git diff*": allow
    "git log*": allow
    "git status": allow
    "git show": allow
  webfetch: deny
```

**Allowed Patterns**:
- `*.review.md` - Any file ending with `.review.md`
- `REVIEW*.md` - Any file starting with `REVIEW`
- `.review/*.md` - Any `.md` file in `.review/` directory

### Subagents (`opencode-agent-security-review.md`, `opencode-agent-performance-review.md`)

```yaml
permission:
  write:
    "*.security.review.md": allow
    "*.performance.review.md": allow
  edit: deny
  bash:
    "git diff*": allow
    "git log*": allow
    "git status": allow
    "git show": allow
  webfetch: deny
```

## How Saving Works

### Main Agent Flow

```python
# 1. Get metadata
timestamp = run("date +'%Y-%m-%d-%H%M%S'")
branch = run("git branch --show-current")

# 2. Generate filename
filename = f".review/uncommitted-{timestamp}-{branch}.md"

# 3. Create review content
review_content = generate_complete_review()

# 4. Save file
write(filename, review_content)

# 5. Inform user
print(f"Review saved to: {filename}")
```

### Example Filename Generation

```bash
# Input
timestamp = "2025-01-13-14:30:22"
branch = "feature-auth"

# Output filename
.review/uncommitted-2025-01-13-143022-feature-auth.md
```

## File Naming Conventions

### Primary Format (Recommended)

```
.review/uncommitted-YYYY-MM-DD-HHMMSS-{branch}.md
```

**Examples**:
- `.review/uncommitted-2025-01-13-143022-main.md`
- `.review/uncommitted-2025-01-13-150115-feature-api.md`
- `.review/uncommitted-2025-01-13-161245-fix-1234.md`

### Alternative Format

```
REVIEW-{branch}-{timestamp}.md
```

**Examples**:
- `REVIEW-main-20250113143022.md`
- `REVIEW-develop-20250113150115.md`

## What Gets Saved

Complete review includes:

1. **Summary** - What files changed and why
2. **Critical Issues** - Must-fix problems
3. **High Priority Issues** - Should-fix problems
4. **Medium Priority Issues** - Nice-to-fix problems
5. **Low Priority Issues** - Optional improvements
6. **Suggestions** - General recommendations
7. **Overall Assessment**
   - Quality Score
   - Security Score (from @security-review)
   - Performance Score (from @performance-review)
   - Ready to commit decision
   - Required actions
8. **Review Metadata**
   - Generated timestamp
   - Git branch
   - Files changed count
   - Review file path
   - Subagent invocation status

## Directory Creation

If `.review/` directory doesn't exist, agent will:

```bash
# Check if directory exists
if [ ! -d ".review" ]; then
    # Create it
    mkdir .review
fi

# Save file to directory
write ".review/uncommitted-{timestamp}.md", content
```

## Benefits of Timestamped Files

1. **History Tracking**: Keep all reviews for reference
2. **Audit Trail**: See code quality over time
3. **Comparison**: Compare reviews across commits
4. **Sharing**: Easily share specific reviews
5. **Documentation**: Permanent record of review decisions

## Review History Management

### View All Reviews

```bash
# List all review files
ls -la .review/

# Show latest review
ls -t .review/ | head -1

# Find reviews for specific branch
ls .review/ | grep "main"

# Find reviews from specific date
ls .review/ | grep "2025-01-13"
```

### Archive Old Reviews

```bash
# Create archive directory
mkdir -p .review/archive

# Move reviews older than 30 days
find .review/ -name "*.md" -mtime +30 -exec mv {} .review/archive/ \;
```

### Delete All Reviews

```bash
# ⚠️ Be careful!
rm -rf .review/
```

## Git Integration

### Should Reviews Be Committed?

**Option 1: Commit reviews** (Recommended for teams)
```bash
# Add .review/ directory
git add .review/

# Commit reviews
git commit -m "Add code review for feature X"

# Push to share with team
git push
```

**Benefits**:
- Team can see reviews
- History preserved in git
- CI/CD can analyze review patterns

**Option 2: Gitignore reviews** (For personal workflows)
```
# Add to .gitignore
.review/
REVIEW*.md
```

**Benefits**:
- Keeps git log clean
- Personal review history only
- No noise in PRs

## Automated Workflows

### Pre-commit Hook (Auto-review & Save)

Create `.git/hooks/pre-commit`:

```bash
#!/bin/bash

echo "🔍 Running code review..."

# Run review agent
opencode run "@uncommitted-review" > /tmp/review_output.txt

# Check for critical issues
if grep -q "## Critical Issues" /tmp/review_output.txt; then
    echo "❌ Critical issues found - aborting commit"
    exit 1
fi

echo "✅ Review passed - committing..."
exit 0
```

### CI/CD Integration

```yaml
# GitHub Actions
- name: Save code review
  run: |
    opencode run "@uncommitted-review"
    
- name: Upload reviews
  uses: actions/upload-artifact@v4
  with:
    name: code-reviews
    path: .review/*.md
```

## Troubleshooting

### File Not Saving

**Symptoms**: Review completes but no file created

**Causes**:
1. Write permissions not loaded (restart OpenCode)
2. Filename pattern doesn't match allowed patterns
3. Directory creation failed

**Solutions**:
1. Restart OpenCode completely
2. Check permissions in agent frontmatter
3. Try saving to project root instead of subdirectory
4. Check file system permissions

### Wrong Filename Format

**Symptoms**: File saved with unexpected name

**Causes**:
1. Timestamp command failing
2. Branch name not captured
3. Pattern not matching correctly

**Solutions**:
1. Check `date` command works: `date +"%Y-%m-%d-%H%M%S"`
2. Check git branch: `git branch --show-current`
3. Use simpler format: `REVIEW-latest.md`

### Permissions Denied

**Symptoms**: Error when trying to save file

**Causes**:
1. Pattern doesn't match `*.review.md`, `REVIEW*.md`, or `.review/*.md`
2. Write permission completely denied

**Solutions**:
1. Verify agent frontmatter permissions
2. Add more permissive pattern: `"*.md": allow`
3. Restart OpenCode to reload permissions

## Best Practices

1. **Always save**: Never complete review without saving
2. **Use consistent naming**: Timestamp + branch format
3. **Keep history**: Don't delete old reviews
4. **Share with team**: Commit reviews if team reviews
5. **Reference in commits**: Link review in commit message
6. **Archive old reviews**: Keep organized over time
7. **Backup reviews**: Add to git or cloud storage

## Example Review File

```markdown
## Summary
Modified 3 files in authentication module adding JWT validation.

## Critical Issues
### src/auth/jwt.py:42 - SQL Injection Vulnerability
- **Description**: Direct string concatenation in query allows injection
- **Impact**: Attackers could bypass authentication
- **Suggestion**: Use parameterized queries
- **Code Example**:
  ```python
  # Bad:
  query = f"SELECT * FROM users WHERE token = '{token}'"
  
  # Good:
  query = "SELECT * FROM users WHERE token = ?"
  cursor.execute(query, (token,))
  ```

## High Priority Issues
### src/auth/session.py:18 - Resource Leak
- **Description**: Database connection not closed on exception
- **Impact**: Connection pool exhaustion under load
- **Suggestion**: Use context manager

## Medium Priority Issues
### src/auth/validator.py:67 - Missing Type Hints
- **Description**: Function returns int but has no annotation
- **Impact**: Reduced IDE support and type checking

## Low Priority Issues
### src/auth/jwt.py:8 - Magic Number
- **Description**: Hardcoded timeout value without explanation
- **Impact**: Maintenance difficulty

## Suggestions
- Add unit tests for new validate_token function
- JWT secret should be in environment variables

## Overall Assessment
- Quality Score: 6/10
- Security Score: 7/10 (from @security-review)
- Performance Score: 8/10 (from @performance-review)
- Ready to commit: No
- Required actions: Fix SQL injection vulnerability and resource leak

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

You can view, share, or reference this file later. Each review creates a new timestamped file to maintain history.
```

## File Size Considerations

### Typical Review Size

- **Small changes** (< 50 lines): 2-5 KB
- **Medium changes** (50-200 lines): 5-15 KB
- **Large changes** (200+ lines): 15-50 KB
- **With subagents**: Add 5-10 KB per subagent

### Storage Management

- **100 reviews**: ~1-5 MB
- **1000 reviews**: ~10-50 MB
- **10,000 reviews**: ~100-500 MB

**Recommendation**: Archive reviews older than 6 months to keep storage manageable.

## Summary

✅ Main agent now automatically saves all reviews
✅ Timestamped filenames for easy tracking
✅ Organized in `.review/` directory
✅ Full metadata included
✅ Ready for git integration
✅ Subagent results integrated
✅ Permission system allows review file creation
