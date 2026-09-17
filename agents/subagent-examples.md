## Example Subagents

This folder contains ready-to-use example subagents:

### 1. Code Reviewer (`code-reviewer.md`)

**Purpose**: Comprehensive code quality and maintainability analysis

**Tools**: Read, Grep, Glob, Bash

**Specialization**:
- Security vulnerability detection
- Performance optimization identification
- Code maintainability assessment
- Test coverage analysis

**Use When**: You need automated code reviews with focus on quality and security

---

### 2. Test Engineer (`test-engineer.md`)

**Purpose**: Test strategy, coverage analysis, and automated testing

**Tools**: Read, Write, Bash, Grep

**Specialization**:
- Unit test creation
- Integration test design
- Edge case identification
- Coverage analysis (>80% target)

**Use When**: You need comprehensive test suite creation or coverage analysis

---

### 3. Documentation Writer (`documentation-writer.md`)

**Purpose**: Technical documentation, API docs, and user guides

**Tools**: Read, Write, Grep

**Specialization**:
- API endpoint documentation
- User guide creation
- Architecture documentation
- Code comment improvement

**Use When**: You need to create or update project documentation

---

### 4. Secure Reviewer (`secure-reviewer.md`)

**Purpose**: Security-focused code review with minimal permissions

**Tools**: Read, Grep

**Specialization**:
- Security vulnerability detection
- Authentication/authorization issues
- Data exposure risks
- Injection attack identification

**Use When**: You need security audits without modification capabilities

---

### 5. Implementation Agent (`implementation-agent.md`)

**Purpose**: Full implementation capabilities for feature development

**Tools**: Read, Write, Edit, Bash, Grep, Glob

**Specialization**:
- Feature implementation
- Code generation
- Build and test execution
- Codebase modification

**Use When**: You need a subagent to implement features end-to-end

---

### 6. Debugger (`debugger.md`)

**Purpose**: Debugging specialist for errors, test failures, and unexpected behavior

**Tools**: Read, Edit, Bash, Grep, Glob

**Specialization**:
- Root cause analysis
- Error investigation
- Test failure resolution
- Minimal fix implementation

**Use When**: You encounter bugs, errors, or unexpected behavior

---

### 7. Data Scientist (`data-scientist.md`)

**Purpose**: Data analysis expert for SQL queries and data insights

**Tools**: Bash, Read, Write

**Specialization**:
- SQL query optimization
- BigQuery operations
- Data analysis and visualization
- Statistical insights

**Use When**: You need data analysis, SQL queries, or BigQuery operations

---

### 8. Clean Code Reviewer (`clean-code-reviewer.md`)

**Purpose**: Readability and maintainability review against clean-code principles

**Tools**: Read, Grep, Glob, Bash

**Specialization**:
- Naming, function length, and argument count
- Duplication and dead code
- Comment quality and intent
- Structural clarity over cleverness

**Use When**: You want a style and maintainability pass distinct from correctness review

---

### 9. Performance Optimizer (`performance-optimizer.md`)

**Purpose**: Identify and remediate performance bottlenecks

**Tools**: Read, Edit, Bash, Grep, Glob

**Specialization**:
- Algorithmic complexity and hot paths
- Memory allocation and leaks
- Caching and query optimization
- Concurrency and I/O bottlenecks

**Use When**: Code is measurably slow and you need targeted optimization

---

## Installation Instructions

### Method 1: Ask Claude (Recommended)

Describe the subagent you want and let Claude create the file:

```text
Create a project-level subagent that runs tests and fixes failures.
Give it access to Bash, Read, Edit, and Grep.
```

Claude writes `.claude/agents/<name>.md` with appropriate frontmatter. Review the generated file, then use it. (The `/agents` interactive creation wizard was removed in v2.1.198 — ask Claude or edit the file directly instead.)

### Method 2: Copy to Project

Copy the agent files to your project's `.claude/agents/` directory:

```bash
# Navigate to your project
cd /path/to/your/project

# Create agents directory if it doesn't exist
mkdir -p .claude/agents

# Copy all agent files from this folder
cp /path/to/04-subagents/*.md .claude/agents/

# Remove the README (not needed in .claude/agents)
rm .claude/agents/README.md
```

### Method 3: Copy to User Directory

For agents available in all your projects:

```bash
# Create user agents directory
mkdir -p ~/.claude/agents

# Copy agents
cp /path/to/04-subagents/code-reviewer.md ~/.claude/agents/
cp /path/to/04-subagents/debugger.md ~/.claude/agents/
# ... copy others as needed
```

### Verification

After installation, verify the agents are recognized by listing the directory:

```bash
ls .claude/agents/
```

You can also ask Claude which subagents are available in the current session, and it will report the built-in and custom agents it can delegate to.

---

## File Structure

```
project/
├── .claude/
│   └── agents/
│       ├── code-reviewer.md
│       ├── test-engineer.md
│       ├── documentation-writer.md
│       ├── secure-reviewer.md
│       ├── implementation-agent.md
│       ├── debugger.md
│       ├── data-scientist.md
│       ├── clean-code-reviewer.md
│       └── performance-optimizer.md
└── ...
```

---
