# START HERE - AI CODE REVIEW EXECUTION PROTOCOL

**MANDATORY EXECUTION ENTRY POINT - AI SYSTEMS ONLY**

---

## 🎯 YOUR ROLE

**You are an expert code analyzer and must apply everything written in this framework to the letter without inventing or skipping anything.**

**Key Principles**:
- Follow ALL rules exactly as written
- Do NOT skip any steps or files
- Do NOT invent information not present in code
- Do NOT deviate from framework methodology
- Complete analysis = 100% adherence to framework

---

## ⛔ ABSOLUTE PROHIBITIONS

**VIOLATION = INVALID ANALYSIS - EXECUTION TERMINATED**

1. ❌ **FORBIDDEN** to start analysis without complete framework documentation reading
2. ❌ **FORBIDDEN** to summarize findings - EVERY finding MUST be listed individually
3. ❌ **FORBIDDEN** to skip files because they "look like tests" or "not important"
4. ❌ **FORBIDDEN** to choose wrong strategy based on LOC count
5. ❌ **FORBIDDEN** to invent or estimate information not verified in code
6. ❌ **FORBIDDEN** to exceed 95% context usage without activating Progressive Writing
7. ❌ **FORBIDDEN** to produce output not conforming to v3.0 Unified Strategy
8. ❌ **FORBIDDEN** to omit validation block (declared_count vs actual_count)
9. ❌ **FORBIDDEN** to aggregate similar findings into single report
10. ❌ **FORBIDDEN** to interpret developer intent not present in code

---

## 🚨 FATAL ERRORS - EXECUTION FAILURES

**The following errors cause IMMEDIATE TERMINATION of analysis:**

### FATAL-001: Incomplete Documentation Reading
- **Condition**: Not all framework documents read before execution
- **Consequence**: Analysis BLOCKED - cannot proceed
- **Recovery**: Complete reading in specified order

### FATAL-002: Wrong Strategy for Codebase Size
- **Condition**: Standard Strategy on codebase >50K LOC
- **Consequence**: Context overflow GUARANTEED - analysis fails
- **Recovery**: Calculate correct LOC, select appropriate strategy

### FATAL-003: Completeness Validation Failed
- **Condition**: `actual_count` outside range `[min_expected, max_expected]` without justification
- **Consequence**: Incomplete analysis - output REJECTED
- **Recovery**: Repeat complete analysis with tracking

### FATAL-004: Summarization Detected
- **Condition**: Output contains "Found N issues" without complete list
- **Consequence**: Completeness violation - output INVALID
- **Recovery**: Rewrite output with ALL findings listed

---

## 🔴 MANDATORY CHECKPOINTS

**Execution MUST stop and await validation at these checkpoints:**

### CHECKPOINT-1: Pre-Analysis Validation
```
REQUIRED BEFORE PROCEEDING:
□ All framework documents read?
□ LOC count executed? (command: find . -name "*.java" | xargs wc -l)
□ Strategy selected based on LOC threshold?
□ Output files initialized (if Progressive Writing)?

IF ANY IS "NO" → BLOCK EXECUTION
```

### CHECKPOINT-2: Strategy Confirmation
```
REQUIRED BEFORE STARTING ANALYSIS:
□ Codebase size: _____ LOC
□ Strategy chosen: STANDARD / HYBRID / PROGRESSIVE
□ Strategy correct for size? (verify table below)

IF STRATEGY WRONG → BLOCK EXECUTION
```

### CHECKPOINT-3: Post-Analysis Validation
```
REQUIRED BEFORE RETURNING OUTPUT:
□ declared_count === actual_count?
□ ALL CRITICAL findings documented?
□ ALL HIGH findings documented?
□ Sampling applied correctly to MEDIUM/LOW?
□ File:line reference present for EVERY finding?

IF ANY IS "NO" → BLOCK OUTPUT, REDO ANALYSIS
```

---

## 📋 MANDATORY READING ORDER - EXECUTION SEQUENCE

**EXECUTE READING IN THIS EXACT ORDER:**

1. **START-HERE.md** (this document - determine strategy)
2. **COMPLETENESS-ENFORCEMENT.md** (3-phase validation - MANDATORY)
3. **UNIVERSAL-CONTEXT-MANAGEMENT.md** (memory management for large codebases)
4. **CLAUDE-ANALYSIS-FRAMEWORK.md** (core workflow)
5. **AGENT-PROMPTS.md** (agent templates with v3.0 output strategy)
6. **FRAMEWORK-RULES-HIERARCHY.md** (conflict resolution)

**AFTER READING:**
- INTERNALIZE completeness rules
- INTERNALIZE output strategy v3.0
- INTERNALIZE validation formulas
- INTERNALIZE context management patterns

**THESE DOCUMENTS ARE EXECUTABLE COMMANDS, NOT SUGGESTIONS.**

---

## EXECUTION PROTOCOL

### Step 1: LOC Count - EXECUTE NOW

```bash
find . -name "*.java" -o -name "*.py" -o -name "*.js" -o -name "*.go" -o -name "*.rb" | xargs wc -l | tail -1
```

### Step 2: Strategy Selection (CRITICAL DECISION POINT)

Based on your assessment, select your execution strategy NOW:

| Codebase Size | Expected Findings | MANDATORY STRATEGY | Consequence of Wrong Choice |
|---------------|-------------------|--------------------|-----------------------------|
| **< 50K LOC** | < 100 issues | **[Standard Strategy](#standard-strategy)** | Fits in memory |
| **50-100K LOC** | 100-200 issues | **[Progressive Writing](#progressive-writing-strategy)** | Recommended for safety |
| **> 100K LOC OR >100 findings OR context >80%** | Any | **[Progressive Writing](#progressive-writing-strategy)** | MANDATORY - will overflow without this |

⚠️ **CRITICAL WARNING**: Choosing wrong strategy causes:
- Standard on large codebase → 32K token overflow ERROR → ANALYSIS FAILS
- Progressive on small codebase → Unnecessary complexity → Wasted time

### Step 3: Mandatory Reading Order

Based on your chosen strategy, read these documents IN THIS EXACT ORDER:

#### Standard Strategy (<50K LOC)
1. **[COMPLETENESS-ENFORCEMENT.md](COMPLETENESS-ENFORCEMENT.md)** - 3-phase validation system (READ FIRST)
2. **[CLAUDE-ANALYSIS-FRAMEWORK.md](CLAUDE-ANALYSIS-FRAMEWORK.md)** - Core workflow (READ SECOND)
3. **[AGENT-PROMPTS.md](AGENT-PROMPTS.md)** - Agent templates (READ THIRD)
4. **DONE** - Proceed to execution

#### Hybrid Strategy (50-100K LOC)
1. **[UNIVERSAL-CONTEXT-MANAGEMENT.md](UNIVERSAL-CONTEXT-MANAGEMENT.md)** - Memory optimization (READ FIRST)
2. **[COMPLETENESS-ENFORCEMENT.md](COMPLETENESS-ENFORCEMENT.md)** - Validation system (READ SECOND)
3. **[CLAUDE-ANALYSIS-FRAMEWORK.md](CLAUDE-ANALYSIS-FRAMEWORK.md)** - Core workflow (READ THIRD)
4. **[AGENT-PROMPTS.md](AGENT-PROMPTS.md)** - Agent templates (READ FOURTH)

#### Progressive Writing Strategy (>100K LOC)
1. **[UNIVERSAL-CONTEXT-MANAGEMENT.md](UNIVERSAL-CONTEXT-MANAGEMENT.md)** - Section 5.6 specifically (READ FIRST)
2. **[CLAUDE-ANALYSIS-FRAMEWORK.md](CLAUDE-ANALYSIS-FRAMEWORK.md)** - Progressive workflow (READ SECOND)
3. **[AGENT-PROMPTS.md](AGENT-PROMPTS.md)** - With write-clear pattern (READ THIRD)
4. **Initialize output files** BEFORE starting analysis:
   ```bash
   # EXECUTE THESE COMMANDS NOW
   touch security_findings.md performance_findings.md
   touch concurrency_findings.md architecture_findings.md
   echo "# Security Findings" > security_findings.md
   echo "# Performance Findings" > performance_findings.md
   echo "# Concurrency Findings" > concurrency_findings.md
   echo "# Architecture Findings" > architecture_findings.md
   ```

### Step 4: Output Format Enforcement (v2.4 - INVARIANT)

**MANDATORY OUTPUT STRATEGY** - This applies to ALL strategies:

```
OUTPUT STRUCTURE (NON-NEGOTIABLE):

1. ALL CRITICAL findings → Detailed format (5 lines each) - NO EXCEPTIONS
2. ALL HIGH findings → Detailed format (5 lines each) - NO EXCEPTIONS
3. 5 MEDIUM findings → Detailed format (representative samples)
4. 5 LOW findings → Detailed format (representative samples)
5. Remaining MEDIUM/LOW → Quick Reference Table (1 line each)
```

**Detailed Format Template**:
```markdown
### [SEC-001] Missing Authorization on Endpoint
**File**: `ConcertiniController.java:38`
**Severity**: CRITICAL
**Problem**: POST endpoint accessible without authentication
**Impact**: Data modification by unauthorized users
**Fix**: Add @PreAuthorize("hasRole('OPERATOR')")
```

**Quick Reference Table Template**:
```markdown
| ID | Severity | Category | File:Line | Brief Description |
|----|----------|----------|-----------|-------------------|
| SEC-051 | MEDIUM | INPUT_VALIDATION | AuthController.java:45 | Missing @Valid annotation |
| SEC-052 | MEDIUM | WEAK_CRYPTO | UserService.java:123 | MD5 used instead of BCrypt |
```

### Step 5: Execution Patterns by Strategy

#### Standard Strategy Pattern
```python
# Pattern: Accumulate in memory, return complete JSON
findings = []
for file in all_files:
    issues = analyze(file)
    findings.extend(issues)
return json.dumps(findings)  # All at once
```

#### Progressive Writing Strategy Pattern
```python
# Pattern: Write incrementally, clear memory, return summary only
with open('security_findings.md', 'a') as f:
    findings_batch = []
    for file in all_files:
        issues = analyze(file)
        findings_batch.extend(issues)

        if len(findings_batch) >= 50:  # Every 50 findings
            f.write(format_findings(findings_batch))
            findings_batch = []  # CLEAR MEMORY - MANDATORY

    # Write remaining
    if findings_batch:
        f.write(format_findings(findings_batch))

# Return SUMMARY ONLY (not findings!)
return {
    "findings_written": 250,
    "file": "security_findings.md"
}
```

### Step 6: Critical Validation Rules

**ENFORCE these rules in ALL strategies**:

1. **No Summarization**: NEVER write "Found 8 SQL injections" without listing all 8 individually
2. **Complete Listing**: EVERY finding MUST have file:line reference
3. **Validation Block**: INCLUDE `declared_count === actual_count` check in output
4. **Progress Reporting**: REPORT every 10% completion

### Execution FAQ

**Q: Unsure about codebase size?**
A: USE Progressive Writing Strategy - it always works and never overflows.

**Q: Can I read all documents regardless of strategy?**
A: YES, but FOLLOW the mandatory order for your chosen strategy.

**Q: What if analysis fails midway?**
A: Progressive Writing saves findings to disk continuously. Standard loses everything.

**Q: Should I use multiple agents?**
A: YES - RUN Security, Performance, Concurrency, and Architecture agents in parallel when possible.

---

## RECOVERY PROCEDURES

### Context Overflow
```bash
# Initialize Progressive Writing immediately
for category in security performance concurrency architecture; do
    echo "# $category Findings" > "${category}_findings.md"
done
```

### Write-Clear Pattern (Progressive Strategy)
```python
with open('output.md', 'a') as f:
    batch = []
    for file in files:
        batch.extend(analyze(file))
        if len(batch) >= 50:
            f.write(format_findings(batch))
            batch = []  # CLEAR MEMORY
```

---

**Framework Version**: 3.0 (AI-Optimized)
**Last Updated**: 2025-10-13

---

END OF EXECUTION PROTOCOL
