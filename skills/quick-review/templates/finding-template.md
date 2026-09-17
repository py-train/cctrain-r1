# Code Review Finding Template

Use this template when documenting issue statistics and themes found during code review.

---

## Finding Statistics (for Reviewer)

When reviewing multiple findings, track:

- **Total Issues Found:** X
- **Critical:** X
- **High:** X
- **Medium:** X
- **Low:** X

**Recommendation:** ✅ Approve / ⚠️ Request Changes / 🔄 Needs Discussion

**Overall Code Quality:** 1-5 stars
(include code complexity metrics in arriving at this rating)

---

## Issues

### Issue Theme: [TITLE]
**Severity|Category**

Choose from the following options:

Severity
- [ ] Critical (blocks deployment)
- [ ] High (should fix before merge)
- [ ] Medium (should fix soon)
- [ ] Low (nice to have)

Category
- [ ] Security
- [ ] Performance
- [ ] Code Quality
- [ ] Maintainability
- [ ] Testing
- [ ] Documentation

eg:
Issue: "Input argument type hints missing or no validation before use"
Medium | Code Quality


#### Representative Code Example
**File:Lines** 
eg: `src/auth/login.py:25-40`

**Function/Method:**
eg: `loginWithKey()`


**Current (Problematic) Code**:
```python
def loginWithKey(key: str, secret: str) -> bool:
    response = login_helper.secureAccess(key)
    ...
```

**Suggested Fix**:
```python
# Validate args
def loginWithKey(key: str, secret: str) -> bool:
    if not key or not secret:
        raise ValueError('Invalid or empty arguments')

    response = login_helper.secureAccess(key)
    ...
});
```

**Related Issues**:
- `login_helper.py:120-125`
- ...

**Reviewer Notes**:
- This is a common pattern in this codebase
- Consider adding this to the code style guide
- Might be worth creating a helper function

---


**Last Updated**: September 4, 2026
**Claude Code Version**: 2.1.220
**Sources**:
- https://code.claude.com/docs/en/skills
**Compatible Models**: Claude Sonnet 5, Claude Sonnet 4.6, Claude Haiku 4.5
