# Universal Code Review Framework v3.0

**A scalable, language-agnostic framework for deep code analysis using AI agents**

**Version**: 3.0
**Last Updated**: 2025-10-12
**Breaking Changes from v2.4**: Pre-Analysis Counting → Estimation, Fixed "5 samples" → Count-based sampling, Sampling threshold >500K LOC (was >100K)

---

## Overview

This framework enables **comprehensive, line-by-line code analysis** of repositories of any size (10K to 500K+ LOC) in any programming language (Java, Python, JavaScript, Go, etc.) using specialized AI agents and intelligent orchestration.

### Key Features (v3.0)

- **Language Agnostic**: Works with Java, Python, JavaScript, and easily extensible to other languages
- **Scalable**: Handles codebases from 10K to 500K+ lines of code (strategic sampling for >500K)
- **Intelligent**: Uses pattern-based scanning to identify hotspots before deep analysis
- **Parallel**: Runs multiple specialized agents concurrently
- **Comprehensive**: Analyzes security, performance, concurrency, resilience, and architecture
- **Factual**: Reports only verified issues with code evidence
- **Actionable**: Provides concrete recommendations with code examples
- **100% Complete**: Guarantees every finding is documented (detailed or in Quick Reference Table)
- **Chain of Thought**: Systematic 6-step analysis with confidence scoring for each finding
- **Validated (v3.0)**: 3-phase enforcement with estimation ranges (not impossible exact counts)
- **Count-Based Sampling (v3.0)**: MEDIUM <20=ALL, LOW <15=ALL (adaptive to finding count)
- **Dynamic Context Management (v3.0)**: Write intervals adapt to context usage (50/25/10/1)
- **Rule Hierarchy (v3.0)**: Clear priority when framework rules conflict (Completeness > Context Mgmt > Output)

### What Problems Does It Solve?

1. **Token Limitations**: Overcomes AI token limits through semantic segmentation
2. **Context Loss**: Maintains system awareness across agent boundaries
3. **Scale**: Analyzes large repositories without missing critical issues
4. **Efficiency**: Pattern scanning identifies hotspots for targeted deep analysis
5. **Completeness**: Ensures 100% code coverage through systematic orchestration
6. **AI Summarization**: Prevents AI from grouping findings ("8 SQL injections found" → lists all 8 with file:line)
7. **Finding Loss (v3.0)**: 3-phase validation with estimation ranges (not impossible exact match)
8. **Framework Contradictions (v3.0)**: Rule hierarchy resolves conflicting directives

---

## Quick Start

### 5-Minute Analysis

```bash
# 1. Clone your repository
cd ~/projects
git clone https://github.com/your-org/your-repo.git
cd your-repo

# 2. Run quick discovery
find . -name "*.java" -o -name "*.py" -o -name "*.js" | wc -l

# 3. Find security hotspots
grep -r "password.*=.*\"" --include="*.java" --include="*.properties" -n | head -10
grep -r "query.*+" --include="*.java" --include="*.py" -n | head -10

# 4. Find performance hotspots
grep -r "\.saveAll(" --include="*.java" -n
grep -r "for.*for.*for" --include="*.java" --include="*.py" -n | head -10
```

### 30-Minute Full Analysis

See [START-HERE.md](START-HERE.md) for the complete framework guide and workflow.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    ORCHESTRATOR                              │
│  • Discovers project structure                              │
│  • Detects languages and frameworks                         │
│  • Generates manifest                                        │
│  • Coordinates agents                                        │
│  • Merges and deduplicates findings                         │
└─────────────────────────────────────────────────────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   SECURITY   │  │ PERFORMANCE  │  │ CONCURRENCY  │
│    AGENT     │  │    AGENT     │  │    AGENT     │
│              │  │              │  │              │
│ • SQL Inject │  │ • N+1 Query  │  │ • Race Cond. │
│ • Auth Gaps  │  │ • Batch Ops  │  │ • Deadlocks  │
│ • XSS/CSRF   │  │ • Algorithms │  │ • Thread Pool│
└──────────────┘  └──────────────┘  └──────────────┘
        │                  │                  │
        └──────────────────┼──────────────────┘
                           ▼
                  ┌──────────────┐
                  │   ASSEMBLY   │
                  │ • Merge      │
                  │ • Dedupe     │
                  │ • Prioritize │
                  └──────────────┘
                           │
                           ▼
                  ┌──────────────┐
                  │    REPORT    │
                  │  • Markdown  │
                  │  • JSON      │
                  └──────────────┘
```

---

## 🤖 FOR AI MODELS: START HERE FIRST

**If you are an AI model tasked with performing code review**:

1. ⚠️ **DO NOT read this README sequentially**
2. ✅ **START FROM [START-HERE.md](START-HERE.md)** - Mandatory reading guide for AI models
3. START-HERE.md will tell you exactly which documents to read and in what order
4. START-HERE.md contains critical decision points for choosing your analysis strategy

**Proceeding without reading START-HERE.md first will cause**:
- ❌ Wrong strategy selection → 32K token overflow errors
- ❌ Skipped critical completeness enforcement rules → summarized findings
- ❌ Missing Progressive Writing implementation → analysis failure on large codebases

**🎯 [→ GO TO START-HERE.md NOW](START-HERE.md)** ← Click here to start correctly

---

## 🎯 Choosing Your Analysis Strategy

**Before starting analysis**, determine which execution strategy to use based on codebase size and expected findings:

| Codebase Size | Expected Findings | Strategy | Reason |
|---------------|-------------------|----------|--------|
| < 50K LOC | < 100 issues | **Standard Output** | Fits comfortably in 32K token output limit |
| 50-100K LOC | 100-200 issues | **Progressive Writing** | May exceed token limit - safer to use incremental writing |
| 100-500K LOC | 200+ issues | **Progressive Writing** ⚠️ **MANDATORY** | Will definitely exceed token limit - must use incremental writing |
| > 500K LOC | 200+ issues | **Progressive Writing + Strategic Sampling** ⚠️ **MANDATORY** | Massive codebase requires 40% minimum sampling |

### Strategy A: Standard Output (Small/Medium Codebases)

**When to use**: Codebase < 100K LOC AND expected findings < 100 issues

**How it works (v3.0)**:
- Agents analyze code and accumulate findings in memory
- Return complete JSON with all findings
- Validate: `actual_count` within `[min_estimate, max_estimate]`
- Maximum output: ~30KB (safe within 32K limit)

**Pros**: Simple, all findings in single response
**Cons**: Fails with 32K overflow if too many findings

### Strategy B: Progressive Writing (Large Codebases) - v3.0

**When to use**: Codebase 100-500K LOC OR expected findings > 100 issues OR when unsure

**How it works**:
1. Each agent writes to **separate category file with Quick Reference Table** during analysis:
   - Security Agent → `security_findings.md` (Quick Reference Table + Detailed Findings)
   - Performance Agent → `performance_findings.md` (Quick Reference Table + Detailed Findings)
   - Concurrency Agent → `concurrency_findings.md` (Quick Reference Table + Detailed Findings)
   - Architecture Agent → `architecture_findings.md` (Quick Reference Table + Detailed Findings)

2. **Write-Clear-Continue pattern**:
   - Analyze findings and write to file immediately
   - Every 50 findings: flush to disk → clear from context
   - Continue analysis with freed memory

3. **v2.4 Output Strategy** (100% documentation):
   - ALL CRITICAL: Detailed format (5 lines each)
   - ALL HIGH: Detailed format (5 lines each)
   - 5 MEDIUM samples: Detailed format (representative examples)
   - 5 LOW samples: Detailed format (representative examples)
   - Remaining MEDIUM/LOW: Quick Reference Table

4. Agent returns **summary only** (2KB instead of 40KB+):
   ```json
   {
     "agent": "Security Agent",
     "output_strategy": "v2.4",
     "findings_found": 250,
     "findings_documented": 250,
     "output_file": "security_findings.md",
     "breakdown": {
       "CRITICAL": {"found": 50, "detailed": 50, "in_table": 0},
       "HIGH": {"found": 32, "detailed": 32, "in_table": 0},
       "MEDIUM": {"found": 143, "detailed": 5, "in_table": 138},
       "LOW": {"found": 25, "detailed": 5, "in_table": 20}
     }
   }
   ```

**Pros**: Never exceeds token limits, scales to 1M+ LOC, 100% findings documented, Quick Reference Tables for navigation
**Cons**: Findings split across multiple files

**⚠️ CRITICAL**: If you have 200+ findings and try Standard Output → 32K TOKEN OVERFLOW ERROR

**✅ When in doubt, use Progressive Writing** (always works, never overflows)

---

## Workflow

### Phase 0: Agent Instruction Briefing (2 minutes)
- Load completeness enforcement rules into agent context
- Configure 3-phase execution (Pre-Analysis Counting → Progressive Extraction → Output Validation)
- Set anti-summarization constraints

### Phase 1: Discovery (5 minutes)
- Scan directory structure
- Detect programming languages
- Identify frameworks (Spring Boot, Django, Express, etc.)
- Count files and lines of code
- Generate project manifest

### Phase 2: Pattern Scanning (5 minutes)
- Use `grep`/`ripgrep` for quick hotspot identification
- Find SQL injection patterns
- Find N+1 query patterns
- Find concurrency issues
- Find hardcoded secrets
- Generate hotspot list

### Phase 3: Agent Execution (30-60 minutes)

Run specialized agents in parallel:
- **Security Agent**: Authentication, authorization, input validation, cryptography
- **Performance Agent**: Database queries, algorithms, caching, transactions
- **Concurrency Agent**: Thread safety, race conditions, deadlocks, resource leaks
- **JPA/Hibernate Agent** (Java): Entity optimization, batch config, lazy loading
- **Resilience Agent**: Timeouts, circuit breakers, retries, bulkheads
- **Architecture Agent**: Dependency violations, coupling, god classes

#### For Large Codebases (>100K LOC) - Progressive Writing Strategy v2.4

Each agent executes with write-clear-continue pattern:

1. **Initialize output file with Quick Reference Table**: `security_findings.md` (category-specific)
2. **Incremental writing loop**:
   - Analyze file and identify findings
   - Write findings to disk immediately (don't accumulate in memory)
   - Every 50 findings: flush to disk → **clear from context** → continue
3. **Apply v2.4 output strategy** (100% documentation):
   - ALL CRITICAL: Detailed format (5 lines each) - no omissions
   - ALL HIGH: Detailed format (5 lines each) - no omissions
   - 5 MEDIUM samples: Detailed format (representative examples)
   - 5 LOW samples: Detailed format (representative examples)
   - Remaining MEDIUM/LOW: Quick Reference Table (ID | Severity | Category | File:Line | Brief Description)
4. **Return summary only** (2KB instead of 40KB+ findings):
   ```json
   {
     "agent": "Security Agent",
     "output_strategy": "v2.4",
     "findings_found": 250,
     "findings_documented": 250,
     "output_file": "security_findings.md",
     "breakdown": {
       "CRITICAL": {"found": 50, "detailed": 50, "in_table": 0},
       "HIGH": {"found": 32, "detailed": 32, "in_table": 0},
       "MEDIUM": {"found": 143, "detailed": 5, "in_table": 138},
       "LOW": {"found": 25, "detailed": 5, "in_table": 20}
     }
   }
   ```

#### For Standard Codebases (<100K LOC) - Traditional Approach

Each agent follows 3-phase execution:

1. **Pre-Analysis Counting**: Declare expected finding count by category
2. **Progressive Extraction**: Report progress every 10% with specific finding IDs
3. **Output Validation**: Return full JSON with all findings, verify declared_count === actual_count

### Phase 4: Result Assembly (5 minutes)
- Merge findings from all agents
- Deduplicate using canonical hashes
- Prioritize by severity (CRITICAL → HIGH → MEDIUM → LOW)
- Generate statistics

### Phase 5: Report Generation (5 minutes)
- Create professional markdown report
- Include code evidence for each finding
- Provide actionable recommendations
- Add quick wins section

### Phase 5.5: Agent Output Validation (Automatic)
- Validate: `declared_count === actual_count`
- Check: No ID gaps, no placeholders, no summarization keywords
- Verify: All required fields present (file, line, code_snippet, description)
- If validation fails: Re-run agent with corrected instructions

**Total Time**: 50-80 minutes for comprehensive analysis

---

## Documentation

### Core Documentation

| Document | Description | For |
|----------|-------------|-----|
| **[START-HERE.md](START-HERE.md)** | **🎯 MAIN ENTRY POINT** - Consolidated guide for both AI models and humans | Everyone |
| [COMPLETENESS-ENFORCEMENT.md](COMPLETENESS-ENFORCEMENT.md) | 3-phase validation system to prevent summarization | AI Models |
| [CLAUDE-ANALYSIS-FRAMEWORK.md](CLAUDE-ANALYSIS-FRAMEWORK.md) | Complete framework methodology with Phase 0-6 workflow | AI Models |
| [AGENT-PROMPTS.md](AGENT-PROMPTS.md) | Agent templates with Chain of Thought reasoning | AI Models |
| [UNIVERSAL-CONTEXT-MANAGEMENT.md](UNIVERSAL-CONTEXT-MANAGEMENT.md) | Memory optimization for large codebases | AI Models |

### Supporting Documentation

| Document | Description | For |
|----------|-------------|-----|
| [EXAMPLES.md](EXAMPLES.md) | Real-world code review examples with detailed findings | Humans |
| [SCRIPTS.md](SCRIPTS.md) | Ready-to-use bash scripts for all analysis tasks | Humans |
| [LANGUAGE-PLUGINS.md](LANGUAGE-PLUGINS.md) | Language-specific patterns for Java, Python, JavaScript | Both |

---

## When to Use This Framework

### Perfect For:
- **New codebases**: Understand inherited or legacy code
- **Pre-deployment audits**: Security and performance review
- **Onboarding**: Technical assessment of new projects
- **Performance investigations**: Find bottlenecks systematically
- **Security audits**: Comprehensive vulnerability scanning
- **Large refactoring**: Identify issues before major changes

### Not Ideal For:
- Single files or small scripts (<500 LOC)
- Real-time CI/CD checks (too thorough, takes 50-80 min)
- Style/formatting only (use linters instead)

---

## Supported Languages

### Fully Supported
- **Java** (Spring Boot, Hibernate, Feign, Maven/Gradle)
- **Python** (Django, Flask, FastAPI, SQLAlchemy)
- **JavaScript/Node.js** (Express, React, Mongoose, Sequelize)

### Easily Extensible
- Go
- Ruby
- PHP
- C#/.NET
- TypeScript
- Kotlin

See [LANGUAGE-PLUGINS.md](LANGUAGE-PLUGINS.md) for adding new languages.

---

## Example Output

### Security Finding
```json
{
  "id": "SEC-CRIT-001",
  "type": "SECURITY",
  "severity": "CRITICAL",
  "category": "SQL_INJECTION",
  "file": "src/main/java/com/example/UserController.java",
  "line": 45,
  "evidence": "String query = \"SELECT * FROM users WHERE email = '\" + email + \"'\";",
  "description": "SQL query constructed using string concatenation with user input",
  "impact": "Attacker can inject arbitrary SQL commands, potentially reading/modifying all database data",
  "recommendation": "Use PreparedStatement:\nString query = \"SELECT * FROM users WHERE email = ?\";\nPreparedStatement stmt = conn.prepareStatement(query);\nstmt.setString(1, email);"
}
```

### Performance Finding
```json
{
  "id": "PERF-CRIT-001",
  "type": "PERFORMANCE",
  "severity": "CRITICAL",
  "category": "N_PLUS_ONE_QUERY",
  "file": "src/main/java/com/example/service/OrderService.java",
  "line": 123,
  "evidence": "List<User> users = userRepository.findAll();\nfor (User user : users) {\n    user.getOrders().size();\n}",
  "description": "N+1 query pattern: loads 500 users then triggers 500 individual queries for orders",
  "impact": "500 database roundtrips instead of 1-2 queries. Operation takes 15 seconds instead of <1 second",
  "recommendation": "Add @BatchSize(size=10) to orders relationship OR use JOIN FETCH:\n@Query(\"SELECT DISTINCT u FROM User u LEFT JOIN FETCH u.orders\")"
}
```

---

## HOW TO READ ANALYSIS RESULTS (v2.4)

After analysis completes, you'll have multiple output files. This section explains how to navigate them effectively and take action on findings.

### Output Structure

**Directory**: `analysis-v2.4/` (or similar)

```
analysis-v2.4/
├── manifest.json                    # Project metadata
├── hotspots.json                    # Pattern scan results
├── CODE_REVIEW_REPORT_v2.4.md      # 🎯 MAIN REPORT (start here)
├── security_findings.md             # Security domain (with Quick Ref Table)
├── performance_findings.md          # Performance domain (with Quick Ref Table)
├── concurrency_findings.md          # Concurrency domain (with Quick Ref Table)
├── jpa_findings.md                  # JPA/Hibernate domain (with Quick Ref Table)
├── resilience_findings.md           # Resilience domain (with Quick Ref Table)
├── architecture_findings.md         # Architecture domain (with Quick Ref Table)
└── findings-all.json                # All findings merged (deduplicated)
```

---

### 1. Start with the Main Report

**File**: `CODE_REVIEW_REPORT_v2.4.md`

**Purpose**: Executive summary + all CRITICAL/HIGH issues detailed

**Key Sections**:
- **Section 1**: Executive Summary (read first - 2 minutes)
- **Section 2**: Project Structure (understand context)
- **Section 3**: CRITICAL & HIGH Issues (100% detailed - act on these first)
- **Section 4**: MEDIUM Issues (5 samples + Quick Reference to rest)
- **Section 5**: LOW Issues (5 samples + Quick Reference to rest)
- **Section 6**: Findings by Domain (navigation index)
- **Section 7**: Statistics
- **Section 8**: Recommendations (prioritized action plan)

**Reading workflow**:

```markdown
1. Read Executive Summary (2 min)
   → Get overview: How many CRITICAL? How many HIGH?
   → Example: "CRITICAL: 8 issues requiring immediate attention"

2. Jump to Section 3: CRITICAL & HIGH Issues
   → ALL detailed (not sampled!)
   → Each finding has:
     • File:line location
     • Problem description (1-2 lines)
     • Impact (concrete consequences)
     • Fix (1-line hint or code example)

3. Triage CRITICAL issues immediately
   → If you see "SQL injection", "missing auth", "thread leak" → FIX NOW
   → Create tickets for each CRITICAL issue

4. Review Section 8: Recommendations
   → Organized by priority: Immediate, Near-Term, Long-Term
   → Focus on "Quick Wins" (high impact, low effort)
```

---

### 2. Understanding Quick Reference Tables (v2.4)

**What are Quick Reference Tables?**

Every domain-specific file (`security_findings.md`, `performance_findings.md`, etc.) starts with a **Quick Reference Table** that indexes **ALL findings** (100%).

**Format**:

| ID | Severity | Category | File:Line | Brief Description |
|----|----------|----------|-----------|-------------------|
| SEC-001 | CRITICAL | SQL_INJ | UserRepo.java:45 | String concatenation in query |
| SEC-002 | HIGH | MISS_AUTH | AdminCtrl.java:23 | No @PreAuthorize on DELETE |
| ... | ... | ... | ... | ... |
| SEC-250 | LOW | WEAK_HASH | UtilService.java:890 | MD5 used instead of SHA256 |

**How to use Quick Reference Tables**:

1. **Quick scan**: Scroll table to see all findings at a glance
2. **Filter by severity**: Look for CRITICAL/HIGH rows first
3. **Filter by category**: Group similar issues (e.g., all SQL_INJ)
4. **Navigate**: Use ID to find detailed finding below

**Example workflow**:

```
1. Open security_findings.md
2. Scan Quick Reference Table (at top)
3. Count CRITICAL: 8 findings
4. Filter table for CRITICAL severity
5. IDs found: SEC-001, SEC-004, SEC-007, SEC-012, SEC-023, SEC-045, SEC-089, SEC-234
6. Scroll down to "Detailed Findings" section
7. Search for "SEC-001" → read full 5-line description
8. Repeat for all 8 CRITICAL findings
```

---

### 3. v2.4 Output Strategy Explained

**Key principle**: ALL CRITICAL + ALL HIGH detailed, rest strategically sampled

#### CRITICAL Findings
- **Coverage**: 100% detailed (no omissions)
- **Format**: 5 lines per finding (File, Severity, Problem, Impact, Fix)
- **Location**:
  - Main report Section 3 (all CRITICAL issues)
  - Domain file "Detailed Findings" section

**Example**:
```markdown
### SEC-001: SQL Injection via String Concatenation
**File**: `UserRepository.java:45`
**Severity**: CRITICAL
**Problem**: Query constructed with string concatenation using user input
**Impact**: Attacker can execute arbitrary SQL (data breach, deletion)
**Fix**: Use PreparedStatement with parameterized queries
```

#### HIGH Findings
- **Coverage**: 100% detailed (no omissions)
- **Format**: Same 5-line format as CRITICAL
- **Location**: Main report Section 3 + domain files

#### MEDIUM Findings
- **Coverage**:
  - 5 representative samples detailed (5 lines each)
  - Remaining in Quick Reference Table (1 line each)
- **Rationale**: Showing 5 examples helps understand the pattern without reading 120+ similar issues

#### LOW Findings
- **Coverage**: Same as MEDIUM (5 samples detailed + rest in Quick Ref Table)
- **Rationale**: Low-priority optimizations that can be addressed in backlog

---

### 4. Navigation Strategies

#### Strategy A: Top-Down (Recommended for First Read)

```
1. Read CODE_REVIEW_REPORT_v2.4.md
   ├─ Section 1: Understand project context
   ├─ Section 3: Absorb all CRITICAL/HIGH (may take 30-60 min)
   ├─ Section 8: Note prioritized recommendations
   └─ Decision: Which issues to fix first?

2. Deep-dive into specific domain
   ├─ Open security_findings.md (if CRITICAL security issues found)
   ├─ Read Quick Reference Table (scan all findings)
   ├─ Read detailed CRITICAL findings (full context)
   └─ Understand patterns (e.g., "8 SQL injections, all in *Repository.java files")

3. Plan fixes
   ├─ Group similar issues (e.g., "all missing @PreAuthorize")
   ├─ Estimate effort (e.g., "15 min per endpoint = 3 hours total")
   └─ Schedule work
```

#### Strategy B: Domain-Specific (For Specialists)

**Use case**: Performance engineer wants to review only performance issues

```
1. Skip main report, go directly to performance_findings.md
2. Read Quick Reference Table
3. Filter for CRITICAL/HIGH in table
4. Read detailed findings for those IDs
5. Implement fixes
```

#### Strategy C: Issue-Specific (For Targeted Fixes)

**Use case**: CTO says "Fix all SQL injection issues immediately"

```
1. Open security_findings.md
2. Find Quick Reference Table
3. Filter table for Category = "SQL_INJ"
4. Get all IDs (e.g., SEC-001, SEC-004, SEC-012)
5. For each ID, find detailed finding
6. Implement fixes following recommendations
7. Verify all SQL_INJ IDs addressed
```

---

### 5. Interpreting Severity Levels

| Severity | Meaning | Timeframe | Example |
|----------|---------|-----------|---------|
| **CRITICAL** | Data breach, system crash, security exploit | **Fix immediately** (same day) | SQL injection, missing auth on admin endpoint, thread leak causing OOM |
| **HIGH** | Significant performance degradation, authentication weakness | **Fix this sprint** (1-2 weeks) | N+1 query (15s response), missing circuit breaker, race condition |
| **MEDIUM** | Code quality issue, minor performance concern | **Fix next sprint** (2-4 weeks) | God class (3000 LOC), missing cache, inefficient algorithm |
| **LOW** | Style improvement, minor optimization | **Backlog** (when convenient) | Unused import, javadoc missing, variable naming |

**Confidence Scoring** (if provided):
- **90%+**: Verified issue, definitely needs fixing
- **70-90%**: Probable issue, verify before fixing
- **50-70%**: Possible issue, needs manual investigation
- **<50%**: Potential false positive, low priority

---

### 6. Post-Analysis Workflow

#### Step 1: Triage (Day 1 - 2 hours)

```
1. Read main report Executive Summary
2. Count CRITICAL issues
3. Create Jira/GitHub issues for each CRITICAL finding
   - Title: "[CRITICAL] {finding title}"
   - Description: Copy from report (File, Problem, Impact, Fix)
   - Priority: P0
   - Assignee: Senior engineer
4. Repeat for HIGH issues (Priority: P1)
```

#### Step 2: Quick Wins (Day 1-2 - 4 hours)

Look for configuration-only fixes (no code changes):

**Examples from report Section 8 "Recommendations"**:
- Add `hibernate.jdbc.batch_size: 25` → application.yml (10 min)
- Reduce Feign timeout from 100s to 10s → application.yml (5 min)
- Add `@PreAuthorize` to 5 endpoints → controllers (30 min)

**Total**: 45 min effort, 3-5x performance improvement

#### Step 3: Sprint Planning (Day 3 - 1 hour)

```
1. Group issues by category
   - All SQL injections → 1 epic
   - All N+1 queries → 1 epic
   - All missing auth → 1 epic

2. Estimate effort
   - CRITICAL: 2-4 hours per issue (senior engineer)
   - HIGH: 1-2 hours per issue (mid-level engineer)
   - MEDIUM: 30 min - 1 hour per issue (junior engineer)

3. Prioritize by impact/effort ratio
   - High impact + low effort = do first
   - High impact + high effort = schedule carefully
   - Low impact + high effort = defer
```

#### Step 4: Execution (Weeks 1-4)

**Sprint 1**:
- Fix all CRITICAL issues (target: 100% resolved)
- Fix top 50% HIGH issues (quick wins)

**Sprint 2**:
- Fix remaining HIGH issues
- Start MEDIUM issues (architectural improvements)

**Sprint 3-4**:
- Continue MEDIUM issues
- Address LOW issues if time permits

#### Step 5: Verification (Ongoing)

After each fix:
1. Run tests
2. Deploy to staging
3. Verify issue resolved
4. Update Jira with "Fixed in commit {hash}"
5. Mark finding ID as "RESOLVED" in tracking sheet

#### Step 6: Re-analysis (Month 2)

```
1. Run framework again on same codebase
2. Compare findings:
   - New issues introduced?
   - Old issues still present?
   - Progress metrics
3. Iterate
```

---

### 7. Common Questions

**Q: How do I know the analysis is complete?**

A: Check these indicators in the report:
- ✅ "Analisi: Completa 100%" in header
- ✅ manifest.json shows all files analyzed
- ✅ All agents completed (security, performance, concurrency, etc.)
- ✅ Validation passed (declared_count === actual_count for each agent)
- ✅ No warnings about skipped files or incomplete domains

**Q: I have 250 findings. Do I need to read all of them?**

A: No! Use the v2.4 prioritization:
1. Read ALL CRITICAL (detailed in Section 3) - may be 5-10 issues
2. Read ALL HIGH (detailed in Section 3) - may be 20-30 issues
3. Scan MEDIUM Quick Reference Table - understand patterns
4. Ignore LOW for now - defer to backlog

**Q: What if I disagree with a severity level?**

A: Adjust based on your context:
- Production system: Hardcoded password = CRITICAL
- Internal tool: Hardcoded password = HIGH
- Proof-of-concept: Hardcoded password = MEDIUM

Update your tracking accordingly.

**Q: How do I share results with my team?**

A: Multiple options:
1. **Exec team**: Send main report Executive Summary (Section 1)
2. **Developers**: Share domain-specific files (security_findings.md, etc.)
3. **PM/PO**: Send Section 8 Recommendations (prioritized action plan)
4. **Architect**: Share findings-all.json for programmatic analysis

**Q: Can I filter findings programmatically?**

A: Yes! Use `findings-all.json`:

```bash
# Extract all CRITICAL findings
cat findings-all.json | jq '.[] | select(.severity=="CRITICAL")'

# Count by severity
cat findings-all.json | jq 'group_by(.severity) | map({severity: .[0].severity, count: length})'

# Filter by file
cat findings-all.json | jq '.[] | select(.file | contains("UserService"))'
```

---

### 8. Tips for Effective Analysis Review

**Do**:
- ✅ Read CRITICAL issues first (always)
- ✅ Group similar findings (e.g., "all N+1 queries")
- ✅ Look for patterns (e.g., "all SQL injection in *Repository files")
- ✅ Use Quick Reference Tables for scanning (fast)
- ✅ Focus on high impact/low effort fixes first ("Quick Wins")
- ✅ Share domain-specific files with specialists
- ✅ Track progress in a spreadsheet or Jira board
- ✅ Re-run analysis after major changes

**Don't**:
- ❌ Read findings in sequential order (waste of time)
- ❌ Try to fix everything at once (burnout)
- ❌ Ignore Quick Reference Tables (they save time!)
- ❌ Skip MEDIUM/LOW entirely (some may be quick fixes)
- ❌ Debate severity levels for hours (adjust and move on)
- ❌ Forget to verify fixes (re-run tests)

---

### 9. Success Metrics

After addressing findings, measure impact:

**Security**:
- CRITICAL security issues: 0 remaining (target: 100% fixed)
- HIGH security issues: < 5 remaining (target: 90% fixed)

**Performance**:
- Response time improvement: measured before/after
- Database query count: measured reduction
- Memory usage: measured decrease

**Code Quality**:
- God classes refactored: tracked count
- Test coverage: measured improvement
- Technical debt hours: measured reduction

**Team Velocity**:
- New features: development speed increase
- Bug fixes: resolution time improvement

---

### 10. Summary

**Quick Reference**:
1. **Start**: CODE_REVIEW_REPORT_v2.4.md → Section 1 (Executive Summary)
2. **Focus**: Section 3 → ALL CRITICAL + HIGH issues (100% detailed)
3. **Quick wins**: Section 8 → Recommendations (prioritized)
4. **Deep dive**: Domain files → Use Quick Reference Tables
5. **Track**: Create tickets, assign, fix, verify
6. **Iterate**: Re-run analysis monthly

**Remember**: v2.4 means **ALL CRITICAL/HIGH documented in detail** (not sampled). You'll never miss a critical issue.

---

## Real-World Results

### Spring Boot Microservice (85K LOC)
- **Analysis Time**: 62 minutes
- **Findings**: 247 issues
  - 9 CRITICAL (3 security, 6 performance)
  - 54 HIGH
  - 142 MEDIUM
  - 42 LOW
- **Quick Wins**: 8 configuration changes = 3-5x performance improvement in 1 hour

### Django Application (45K LOC)
- **Analysis Time**: 38 minutes
- **Findings**: 128 issues
  - 5 CRITICAL (4 security, 1 performance)
  - 32 HIGH
  - 68 MEDIUM
  - 23 LOW
- **Key Discoveries**: SQL injection, N+1 queries, missing authentication

### Node.js API (32K LOC)
- **Analysis Time**: 29 minutes
- **Findings**: 89 issues
  - 3 CRITICAL (XSS, missing auth, event loop blocking)
  - 24 HIGH
  - 45 MEDIUM
  - 17 LOW
- **Impact**: Fixed 3 CRITICAL issues in 2 hours

---

## Key Concepts

### Completeness Enforcement (NEW in v2.1)
**3-phase validation system** guarantees 100% finding documentation:
- **Phase 1**: Pre-Analysis Counting - Agent declares expected finding count before analyzing
- **Phase 2**: Progressive Extraction - Agent reports progress every 10% with specific IDs
- **Phase 3**: Output Validation - Verify `declared_count === actual_count`

**Anti-Summarization**: Prevents "Found 8 SQL injections" → Forces listing all 8 individually with file:line.

### Chain of Thought Reasoning (NEW in v2.1)
Each finding analyzed through **6-step systematic process**:
1. **Observation**: What code pattern exists?
2. **Analysis**: Why is this problematic?
3. **Context**: What makes it exploitable/problematic?
4. **Impact**: What are the consequences?
5. **Alternative Explanations**: Could this be a false positive?
6. **Confidence**: 90%+, 70-90%, 50-70%, or <50%

### Role-Based Agent Personas (NEW in v2.1)
Each agent has specific identity and expertise:
- **Alex "Paranoid" Rodriguez** - Security Agent (12+ years AppSec)
- **Maria "Profiler" Chen** - Performance Agent (15+ years DB optimization)
- **David "Parallel" Kumar** - Concurrency Agent (10+ years race conditions)
- **Sarah "ORM Whisperer" Patel** - JPA/Hibernate Agent (12+ years Hibernate)
- **James "Failover" Martinez** - Resilience Agent (10+ years fault-tolerance)
- **Emily "Architect" Zhang** - Architecture Agent (15+ years clean architecture)

### Semantic Segmentation
Split code by **architectural layers** (controller, service, DAO) rather than arbitrary file counts. Maintains context and reduces token usage.

### Context Injection
Provide minimal cross-layer context to agents (e.g., service knows what controllers call it). Prevents losing system-wide awareness.

### Pattern-Based Scanning
Use `grep` patterns to identify hotspots before deep analysis. Reduces 845 files to 50 files requiring detailed review.

### Tiered Analysis
- **Tier 1** (Quick): Pattern matching, metrics (all files, 0 tokens)
- **Tier 2** (Standard): Agent reads file, checks common issues (50% of files)
- **Tier 3** (Deep): Line-by-line analysis (10% of files - hotspots only)

### Language Plugins
Core framework is language-agnostic. Language-specific patterns and rules loaded as plugins.

### Hash-Based Deduplication
Use `hash(file+line+category)` to identify duplicate findings from multiple agents.

---

## Tactical Approaches

1. **Semantic Segmentation**: Divide by architecture, not file count
2. **Context Injection**: Maintain cross-layer awareness
3. **Pattern Scanning**: Grep first, analyze deep second
4. **Tiered Analysis**: Adaptive depth based on risk
5. **Language Plugins**: Core + language-specific modules
6. **Deduplication**: Canonical hashing of findings
7. **Cross-Cutting Agents**: Agents that span multiple layers
8. **Dependency Graphs**: Understand call relationships
9. **Size-Based Routing**: Different strategies for different file sizes
10. **Parallel Execution**: Run independent agents concurrently

See [CLAUDE-ANALYSIS-FRAMEWORK.md](CLAUDE-ANALYSIS-FRAMEWORK.md) for detailed explanations.

---

## Advanced Usage

### Custom Agents

Add domain-specific agents:

```markdown
# File: prompts/api-design-agent-prompt.md

## Mission
Analyze REST API design for consistency, versioning, and best practices.

## Checks
- Consistent naming (camelCase vs snake_case)
- Proper HTTP status codes
- Pagination implemented
- API versioning strategy
- Rate limiting
```

### CI/CD Integration

```yaml
# .github/workflows/code-review.yml
name: Deep Code Review

on:
  pull_request:
    branches: [main]

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run analysis
        run: |
          ./scripts/discover.sh
          ./scripts/pattern-scan.sh
          claude-code run-agent --prompt security-prompt.md
          ./scripts/generate-report.sh
      - name: Upload report
        uses: actions/upload-artifact@v2
        with:
          name: code-review-report
          path: CODE_REVIEW_REPORT.md
```

### Custom Language Support

```yaml
# plugins/go/patterns.yaml
security:
  - "password.*="
  - "exec.Command.*+"
  - "sql.Query.*+"

performance:
  - "for.*range"
  - "append("

concurrency:
  - "go func"
  - "sync.Mutex"
  - "chan "
```

---

## Best Practices

### Do:
- ✅ Run discovery phase first
- ✅ Use pattern scanning to identify hotspots
- ✅ Run agents in parallel when possible
- ✅ Review CRITICAL and HIGH findings first
- ✅ Provide code evidence for every finding
- ✅ Include actionable recommendations
- ✅ Track findings over time
- ✅ **Count findings before analyzing** (Pre-Analysis Counting phase)
- ✅ **Report progress every 10%** during extraction
- ✅ **List every finding individually** - never group or summarize
- ✅ **Include validation block** in output JSON
- ✅ **Use Chain of Thought reasoning** for each finding

### Don't:
- ❌ Skip discovery - you'll miss context
- ❌ Analyze entire codebase without pattern scan
- ❌ Run agents sequentially (waste time)
- ❌ Report findings without code evidence
- ❌ Ignore framework-specific optimizations
- ❌ Assume technologies not explicitly found
- ❌ Create findings based on opinions
- ❌ **Summarize findings** (e.g., "8 SQL injections found" - list all 8!)
- ❌ **Skip pre-analysis counting** - declare expected finding count first
- ❌ **Omit validation block** - always include validation in output JSON

---

## Limitations

1. **Time**: Comprehensive analysis takes 50-80 minutes
2. **Accuracy**: AI agents can have false positives (verify findings)
3. **Context**: Some business logic requires human understanding
4. **Coverage**: Doesn't replace security penetration testing
5. **Languages**: Not all languages equally supported (extendable)

---

## Contributing

This framework is designed to be extended. Contributions welcome:

1. **New language plugins**: Add patterns for Go, Ruby, PHP, etc.
2. **New agent types**: Monitoring, observability, scalability agents
3. **Improved patterns**: More accurate grep patterns
4. **Orchestration scripts**: Better automation
5. **Integration examples**: CI/CD, IDEs, Git hooks

---

## FAQ

### How is this different from static analysis tools?

Static analysis tools (SonarQube, ESLint, etc.) are:
- Fast (seconds to minutes)
- Pattern-based (predefined rules)
- Language-specific
- Integrated into CI/CD

This framework:
- Thorough (50-80 minutes)
- AI-powered (understands context)
- Language-agnostic
- Best for comprehensive audits

**Use both**: Static analysis for continuous checks, this framework for periodic deep dives.

### Can I use this with proprietary code?

Yes. All analysis happens locally. No code is sent to external services except the AI API (Claude Code).

### How accurate are the findings?

- **CRITICAL findings**: ~95% accuracy (verified issues)
- **HIGH findings**: ~85% accuracy
- **MEDIUM/LOW**: May include false positives, requires human review

Always verify CRITICAL findings before acting.

### Can I customize severity levels?

Yes. Severity depends on context:
- Production system: Hardcoded password = CRITICAL
- Internal tool: Hardcoded password = HIGH
- Proof-of-concept: Hardcoded password = MEDIUM

Adjust in agent prompts.

### How do I add a new programming language?

1. Create `plugins/[language]/` directory
2. Add `patterns.yaml` with grep patterns
3. Add language-specific agent logic
4. Update detection in discovery phase
5. Test with sample repository

See [LANGUAGE-PLUGINS.md](LANGUAGE-PLUGINS.md) for template.

### What is the Completeness Enforcement mechanism?

The **3-phase validation system** that guarantees 100% finding documentation:

1. **Phase 1 - Pre-Analysis Counting**: Agent must count and declare expected findings BEFORE analyzing
2. **Phase 2 - Progressive Extraction**: Agent reports progress every 10% with specific finding IDs
3. **Phase 3 - Output Validation**: Automated check ensures `declared_count === actual_count`

This prevents AI from summarizing findings (e.g., "8 SQL injections found" without listing all 8).

### What is Chain of Thought reasoning?

Each finding is analyzed through a **6-step systematic process**:

1. **Observation**: What code pattern exists?
2. **Analysis**: Why is this problematic?
3. **Context**: What makes it exploitable/problematic?
4. **Impact**: What are the consequences?
5. **Alternative Explanations**: Could this be a false positive?
6. **Confidence**: Assign 90%+, 70-90%, 50-70%, or <50% confidence level

This ensures thorough, factual analysis with clear reasoning for each finding.

---

## License

This framework documentation is provided as-is for educational and professional use.

---

## Support

- **Documentation**: See files in this repository
- **Issues**: Create GitHub issue
- **Discussions**: GitHub Discussions
- **Examples**: See [EXAMPLES.md](EXAMPLES.md)

---

## Acknowledgments

Built with insights from analyzing:
- Spring Boot microservices (50K-100K LOC)
- Django applications (20K-50K LOC)
- Node.js APIs (15K-40K LOC)
- Legacy enterprise systems (100K+ LOC)

Special thanks to the Claude Code team for the powerful agent orchestration capabilities.

---

**Version**: 2.4
**Last Updated**: 2025-10-12
**Maintained By**: Code Review Framework Community

### What's New in v2.4

- 🆕 **Quick Reference Tables**: Every agent file starts with navigable table of ALL findings (100% indexed)
- 🆕 **Complete CRITICAL/HIGH Coverage**: ALL critical and high findings in detailed format (not sampled)
- 🆕 **Enhanced Output Strategy**: ALL CRITICAL + ALL HIGH detailed + 5 MEDIUM + 5 LOW samples + Quick Reference Table for rest
- 🆕 **100% Documentation**: No findings omitted - every single issue preserved (0% omission rate)
- 🆕 **Structured Navigation**: Quick Reference format (ID | Severity | Category | File:Line | Brief Description)
- 🆕 **Agent Output Format**: Mandatory Quick Reference Table at top + Detailed Findings sections
- 🆕 **Improved Summary**: Breakdown includes `detailed` vs `in_table` counts for transparency

### What's New in v2.3

- 🚀 **Progressive Writing Strategy**: Write findings to disk DURING analysis, not at end
- 🚀 **32K Output Limit Solution**: Incremental write-clear-continue pattern bypasses token limits
- 🚀 **Scalability Proven**: Successfully analyzed 138K+ LOC (1,350 files) without overflow
- 🚀 **Constant Memory Usage**: Write every 50 findings → clear from context → 67% savings
- 🚀 **Intelligent Sampling**: CRITICAL=ALL, HIGH=ALL, MEDIUM=30%, LOW=20%
- 🚀 **Agent Architecture**: Each agent writes to separate file (security_findings.md, performance_findings.md, etc.)
- 🚀 **Summary-Only Returns**: Agents return 2KB summary instead of 40KB+ findings

### What's New in v2.2

- 🆕 **Universal Context Management**: Smart in-memory compression for large codebases
- 🆕 **Adaptive Strategies**: Automatic mode selection based on codebase size (10K-100K+ LOC)
- 🆕 **Progressive Compression**: Full details → Compressed → Pattern codes based on context usage
- 🆕 **Equal Domain Priority**: 25% each for Security, Performance, Concurrency, Architecture
- 🆕 **Smart Sampling**: Strategic 60% coverage for very large codebases (>100K LOC)
- 🆕 **Context Monitoring**: Active threshold management (0-60%, 60-70%, 70-80%, 80-90%, >90%)
- 🆕 **Intelligent Batching**: Group similar findings to save context without losing quality

### What's New in v2.1

- ✨ **START-HERE.md**: Mandatory reading guide for AI models
- ✨ **Completeness Enforcement**: 3-phase validation system preventing summarization
- ✨ **Chain of Thought Reasoning**: 6-step systematic analysis for each finding
- ✨ **Agent Personas**: Role-based identities with specific expertise
- ✨ **Output Validation**: Automated check ensuring `declared_count === actual_count`
- ✨ **Bash Toolkits**: Efficient grep patterns for each agent
- ✨ **Confidence Scoring**: 4-tier confidence levels for all findings

---

## Getting Started

### For AI Models
1. **Read [START-HERE.md](START-HERE.md) FIRST** - Mandatory reading order for AI
2. Read [COMPLETENESS-ENFORCEMENT.md](COMPLETENESS-ENFORCEMENT.md) - Learn 3-phase validation
3. Review [CLAUDE-ANALYSIS-FRAMEWORK.md](CLAUDE-ANALYSIS-FRAMEWORK.md) - Overall workflow
4. Study [AGENT-PROMPTS.md](AGENT-PROMPTS.md) - Agent templates and examples
5. **Read [UNIVERSAL-CONTEXT-MANAGEMENT.md](UNIVERSAL-CONTEXT-MANAGEMENT.md) for large codebases (>50K LOC)** - Context optimization strategies
6. ⚠️ **CRITICAL DECISION POINT**:
   - **IF** codebase > 100K LOC **OR** expecting > 100 findings:
     - ✅ **YOU MUST** use Progressive Writing Strategy (see Step 5.6 in START-HERE.md)
     - ✅ **YOU MUST** initialize separate output files per agent (security_findings.md, etc.)
     - ✅ **YOU MUST** return summary only (NOT full findings in response)
     - ⛔ **DO NOT** attempt standard output (will cause 32K token overflow ERROR)
   - **ELSE**: Standard output with full JSON is acceptable

### For Humans
1. Read [START-HERE.md](START-HERE.md) for getting started
2. Review [CLAUDE-ANALYSIS-FRAMEWORK.md](CLAUDE-ANALYSIS-FRAMEWORK.md) for methodology
3. Check [AGENT-PROMPTS.md](AGENT-PROMPTS.md) for prompt templates
4. Explore [LANGUAGE-PLUGINS.md](LANGUAGE-PLUGINS.md) for language-specific patterns
5. Run your first analysis!

**Happy analyzing!** 🚀
