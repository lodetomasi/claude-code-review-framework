# START HERE - Universal Code Review Framework Guide

**Version**: 3.0
**Last Updated**: 2025-10-12
**Audience**: AI Models (Claude, GPT, etc.) and Human Developers
**Breaking Changes from v2.4**:
- Pre-Analysis Counting → Estimation with confidence intervals
- Fixed "5 samples" → Count-based sampling (MEDIUM <20=ALL, LOW <15=ALL)
- Sampling threshold: >500K LOC (was >100K LOC)

---

## 🚀 Quick Navigation

Choose your path based on your role and needs:

| You Are | Time Available | Start Here |
|---------|----------------|------------|
| **Human Developer** | 5 minutes | [→ 5-Minute Quick Scan](#5-minute-quick-scan) |
| **Human Developer** | 30+ minutes | [→ Full Analysis Workflow](#full-analysis-workflow) |
| **AI Model** | Any | [→ AI Model Instructions](#ai-model-instructions-mandatory) |
| **Looking for Examples** | Any | [→ EXAMPLES.md](EXAMPLES.md) |
| **Need Scripts** | Any | [→ SCRIPTS.md](SCRIPTS.md) |
| **Need Help** | Any | [→ Troubleshooting](#troubleshooting) |

---

## BEFORE YOU START - Critical Assessment

### 1. Codebase Size Assessment
Check ONE option that matches your target codebase:

- [ ] **< 50K LOC** → Read Steps 1-4, use Standard Output
- [ ] **50-100K LOC** → Read Steps 1-5.5, consider Progressive Writing
- [ ] **100-500K LOC** → Read Steps 1-5.6 **MANDATORY**, use Progressive Writing
- [ ] **> 500K LOC** → Read Steps 1-5.6 **MANDATORY**, use Progressive Writing + Strategic Sampling (40% minimum)

### 2. Expected Findings Assessment
Estimate from pattern scan results:

- [ ] **< 50 issues** → Standard output acceptable
- [ ] **50-100 issues** → Progressive Writing recommended
- [ ] **> 100 issues** → Progressive Writing **MANDATORY**

### 🚨 CRITICAL DECISION RULES

**IF you checked ANY rightmost option above**:
- ⛔ **DO NOT** skip Step 5.6 (Progressive Writing Strategy)
- ⛔ **DO NOT** attempt standard output (will cause 32K token overflow ERROR)
- ✅ **YOU MUST** use write-clear-continue pattern
- ✅ **YOU MUST** write to separate files per agent
- ✅ **YOU MUST** return summary only (not full findings)

**IF unsure**: Use Progressive Writing (always works, never fails)

**Consequences of wrong choice**:
- 200+ findings with standard output → **32,000 TOKEN OVERFLOW ERROR** 💥
- 50 findings with progressive writing → Works but unnecessary overhead ✅

**Golden rule**: **When in doubt, use Progressive Writing**

---

## 📖 READING ORDER (MANDATORY)

### Step 1: Read This File First
**File**: `START-HERE.md` (current file)
**Time**: 2 minutes
**Purpose**: Understand the reading sequence

---

### Step 2: Understand Completeness Enforcement (v3.0)
**File**: `COMPLETENESS-ENFORCEMENT.md`
**Time**: 5 minutes
**Purpose**: Learn the **THREE-PHASE** process that prevents you from summarizing findings

**Why Critical**: Without reading this, you WILL make the mistake of writing:
- ❌ "Found 8 SQL injection vulnerabilities"
- ✅ Instead of listing all 8 individually with file:line

**Key Concepts (v3.0)**:
- Phase 1: Pre-Analysis ESTIMATION (confidence intervals [min, max])
- Phase 2: Progressive Extraction (10% checkpoints)
- Phase 3: Output Validation (range-based, not exact match)

**After reading, you MUST**:
- Always estimate finding count range BEFORE analyzing (not exact count)
- Report progress every 10%
- Output JSON with `analysis_metadata` containing estimated_range and actual_count
- Validate: actual_count within [min_estimate, max_estimate] OR document variance

---

### Step 3: Learn the Overall Framework
**File**: `CLAUDE-ANALYSIS-FRAMEWORK.md`
**Time**: 10 minutes
**Purpose**: Understand the 6-phase workflow

**Key Sections**:
1. **Phase 0**: Agent Instruction Briefing (COMPLETENESS RULES)
2. **Phase 1**: Discovery (manifest generation)
3. **Phase 2**: Pattern Scanning (grep hotspots)
4. **Phase 3**: Parallel Agent Execution
5. **Phase 5.5**: Agent Output Validation (NEW!)
6. **Phase 6**: Report Generation

**What You'll Learn**:
- How to use semantic segmentation (not arbitrary chunks)
- How to inject context between agents
- Token budget allocation (200k total)
- Language plugin system
- Deduplication algorithm

---

### Step 4: Study Agent Prompt Templates
**File**: `AGENT-PROMPTS.md`
**Time**: 15 minutes
**Purpose**: Learn HOW to execute each specialized agent

**Read in This Order**:

1. **UNIVERSAL AGENT CONTEXT BLOCK** (lines 7-86)
   - Output format with metadata/validation
   - Severity guidelines
   - Constraints

2. **COMPLETENESS ENFORCEMENT RULES** (lines 80-226)
   - Three-phase execution mandatory
   - Anti-summarization examples
   - Rejection criteria

3. **Agent Templates** (pick based on language):
   - Security Agent (lines 230-463)
   - Performance Agent (lines 450-705)
   - Concurrency Agent (lines 724-1020)
   - JPA/Hibernate Agent (lines 1024-1312) - Java only
   - Resilience Agent (lines 1317-1600)

**For Each Agent, Study**:
- Mission statement
- Analysis checklist
- Language-specific checks
- Output examples

---

### Step 5.4 (NEW v3.0): Framework Rules Hierarchy
**File**: `FRAMEWORK-RULES-HIERARCHY.md`
**Time**: 5 minutes
**Purpose**: Understand rule priority when conflicts arise
**READ BEFORE executing agents to resolve contradictions**

**Why Critical**: Framework v2.4 had conflicting rules. v3.0 defines priority hierarchy.

**3-Level Hierarchy**:
1. **PRIORITY 1: COMPLETENESS** (find all issues)
2. **PRIORITY 2: CONTEXT MANAGEMENT** (compression technique)
3. **PRIORITY 3: OUTPUT STRATEGY** (presentation format)

**Conflict Resolution Example**:
- Q: "I found 80 MEDIUM findings. Do I document all or sample?"
- A: Priority 1 says find all (✅), Priority 2 says compress during analysis (✅), Priority 3 says present top 5 detailed + table (✅)
- Result: Find all 80, write all 80 to disk, present 5 detailed + 75 in Quick Ref Table

**Key Insight**: Completeness is about FINDING all issues, not PRESENTING all in full detail.

---

### Step 5.5 (CRITICAL - v3.0): Universal Context Management
**File**: `UNIVERSAL-CONTEXT-MANAGEMENT.md`
**Time**: 8 minutes
**Purpose**: Master smart compression for large codebases
**READ BEFORE analyzing codebases >50K LOC**

**Why Critical**: Context window is limited (200K tokens). Large projects need intelligent compression.

**Key Principles (v3.0)**:
1. **Monitor context actively** (check every 10 files)
2. **Equal domain priority** (25% Security, 25% Performance, 25% Concurrency, 25% Architecture)
3. **Progressive compression** (during analysis only, expand for output)
4. **Dynamic write intervals** (50/25/10/1 based on context usage 70%/85%/95%)
5. **Strategic sampling** (only for >500K LOC codebases, minimum 40%)

**Context Thresholds**:
```
0-60%: Full analysis
60-70%: Start batching similar findings
70-80%: CRITICAL + HIGH only
80-90%: CRITICAL only with pattern codes
>90%: Emergency output
```

**Adaptive Strategies (v3.0)**:
- **<10K LOC**: Single pass, full details
- **10-50K LOC**: Layer-based chunks
- **50-100K LOC**: Progressive Writing, full analysis (100%)
- **100-500K LOC**: Progressive Writing, full analysis (100%)
- **>500K LOC**: Strategic sampling (40% minimum coverage)

**Memory Management**:
```
ANALYZE → EXTRACT → COMPRESS → CLEAR

Keep: CRITICAL findings with full context
Clear: File contents, duplicate patterns, boilerplate
```

**Output Reconstruction**:
- During analysis: Store compressed (save context)
- Final output: Expand to full detail (user sees complete findings)

**Read Full Document**: `UNIVERSAL-CONTEXT-MANAGEMENT.md` for:
- Complete threshold guidelines
- Compression format examples
- Batching algorithms
- Sampling strategies
- Pattern libraries

---

### Step 5.6 (⚠️ MANDATORY for >100K LOC - v3.0): Progressive Writing Strategy
**File**: `UNIVERSAL-CONTEXT-MANAGEMENT.md` (Section: Progressive Writing v3.0)
**Time**: 5 minutes
**Purpose**: Solve the 32K output token limit for large-scale analysis
**⚠️ READ BEFORE analyzing codebases with >100K LOC or 100+ expected findings**

**Why Critical**: Claude has a 32,000 token OUTPUT limit (~40KB). Large analyses generate 800+ findings = 50KB+ output → OVERFLOW ERROR.

**The Problem**:
```
Agent finds 250 issues → tries to return all in JSON → exceeds 32K token limit → ERROR
```

**The Solution - Write-Clear-Continue Pattern (v3.0 with Dynamic Intervals)**:
```bash
findings_count=0

# v3.0: Dynamic write interval based on context usage
context_usage=$(get_context_usage_percentage)

if [ "$context_usage" -lt 70 ]; then
    BATCH_SIZE=50
elif [ "$context_usage" -lt 85 ]; then
    BATCH_SIZE=25
elif [ "$context_usage" -lt 95 ]; then
    BATCH_SIZE=10
else
    BATCH_SIZE=1  # Write immediately if >95%
fi

# CLEAR from context when threshold reached
if [ $((findings_count % BATCH_SIZE)) -eq 0 ]; then
    echo "[Progress] $findings_count findings written (interval: $BATCH_SIZE)"
    # Context freed - can continue with constant memory
fi
```

**Agent Architecture (v3.0)**:
- Each agent writes to **separate category file** with **Quick Reference Table**:
  - Security Agent → `security_findings.md` (with Quick Reference Table at top)
  - Performance Agent → `performance_findings.md` (with Quick Reference Table at top)
  - Concurrency Agent → `concurrency_findings.md` (with Quick Reference Table at top)
  - Architecture Agent → `architecture_findings.md` (with Quick Reference Table at top)

**Key Benefits (v3.0)**:
- ✅ **Constant Memory**: Write → Clear → Context stays at ~50KB regardless of findings
- ✅ **No Output Overflow**: Return 2KB summary instead of 40KB+ findings
- ✅ **Scales to 500K+ LOC**: Write 10,000 findings without hitting limits
- ✅ **100% Documented**: ALL findings preserved (detailed or in Quick Reference Table)
- ✅ **Quick Navigation**: Quick Reference Tables for instant location lookup
- ✅ **Count-Based Sampling**: MEDIUM <20=ALL, LOW <15=ALL (adaptive to finding count)
- ✅ **Dynamic Intervals**: 50/25/10/1 based on context usage (70%/85%/95% thresholds)
- ✅ **Pre-Analysis Estimation**: Confidence intervals [min, max] instead of exact counts
- ✅ **67% Context Savings**: Proven on 138K LOC project (464 findings)

**v3.0 Unified Output Strategy (Count-Based)**:
- **ALL CRITICAL**: Detailed format (5 lines each) - never sampled
- **ALL HIGH**: Detailed format (5 lines each) - never sampled
- **MEDIUM samples**: Count-based rules
  - <20 total: ALL detailed
  - 20-50 total: Top 10 detailed + Quick Reference Table
  - >50 total: Top 5 detailed + Quick Reference Table
- **LOW samples**: Count-based rules
  - <15 total: ALL detailed
  - 15-40 total: Top 8 detailed + Quick Reference Table
  - >40 total: Top 3 detailed + Quick Reference Table
- **Quick Reference Table**: 1 line each (ID, severity, category, file:line, brief description)

**When to Use**:
- Codebase 100-500K LOC (full analysis with Progressive Writing)
- Codebase >500K LOC (strategic sampling minimum 40%)
- Expected findings >100 issues
- Multiple parallel agents
- Deep comprehensive analysis

**Read Full Section**: `UNIVERSAL-CONTEXT-MANAGEMENT.md` lines 425-660 for:
- Complete implementation with bash examples
- Incremental writing patterns
- Sampling algorithms
- Context monitoring during writes

---

## Part 1: 5-Minute Quick Scan (For Humans)

Get actionable insights in 5 minutes or less.

### Prerequisites
```bash
# Check you have required tools
command -v find && command -v grep && command -v wc || echo "Missing required tools"
```

### Quick Discovery Script
```bash
#!/bin/bash
# Save as: quick-scan.sh

echo "=== Repository Quick Scan ==="
echo "Date: $(date)"
echo ""

# Language detection
echo "=== Languages Detected ==="
find . -name "*.java" 2>/dev/null | head -1 && echo "✓ Java"
find . -name "*.py" 2>/dev/null | head -1 && echo "✓ Python"
find . -name "*.js" 2>/dev/null | head -1 && echo "✓ JavaScript"
find . -name "*.go" 2>/dev/null | head -1 && echo "✓ Go"

# Framework detection
echo ""
echo "=== Frameworks Detected ==="
test -f pom.xml && echo "✓ Maven (Java)"
test -f build.gradle && echo "✓ Gradle (Java)"
test -f requirements.txt && echo "✓ Python/pip"
test -f package.json && echo "✓ Node.js/npm"
test -f go.mod && echo "✓ Go modules"

# Size assessment
echo ""
echo "=== Codebase Size ==="
echo "Total files: $(find . -type f -name "*.java" -o -name "*.py" -o -name "*.js" 2>/dev/null | wc -l)"
echo "Total LOC: $(find . -type f \( -name "*.java" -o -name "*.py" -o -name "*.js" \) -exec wc -l {} + 2>/dev/null | tail -1 | awk '{print $1}')"

# Critical security issues
echo ""
echo "=== Security Hotspots (Top 5) ==="
echo "Hardcoded passwords:"
grep -r "password.*=.*['\"]" --include="*.java" --include="*.py" --include="*.js" 2>/dev/null | head -5

echo ""
echo "SQL injection risks:"
grep -r "query.*+" --include="*.java" --include="*.py" 2>/dev/null | head -5

# Performance issues
echo ""
echo "=== Performance Hotspots (Top 5) ==="
echo "N+1 query patterns:"
grep -r "for.*:" --include="*.java" --include="*.py" -A 2 2>/dev/null | grep -E "get[A-Z]|find|query" | head -5

echo ""
echo "=== Quick Scan Complete ==="
```

### Run It Now
```bash
chmod +x quick-scan.sh
./quick-scan.sh | tee quick-scan-results.txt
```

**What You Get in 5 Minutes**:
- Language and framework identification
- Codebase size assessment
- Top security vulnerabilities
- Major performance issues
- Actionable hotspots to investigate

For more scripts, see [SCRIPTS.md](SCRIPTS.md)

---

## Part 2: AI Model Instructions (MANDATORY)

### ⚠️ STOP - Assessment Required

**Before proceeding, you MUST assess the codebase to choose the correct strategy.**

### Step 1: Codebase Assessment Checklist

Execute this assessment NOW:

```markdown
ASSESSMENT CHECKLIST:
□ Codebase size: _____ LOC (use find/wc commands)
□ Estimated findings: _____ issues (based on patterns found)
□ Available memory: Standard Claude context window
□ Output format needed: JSON / Markdown / Both
```

### Step 2: Strategy Selection (CRITICAL DECISION POINT)

Based on your assessment, choose your execution strategy:

| Codebase Size | Expected Findings | YOU MUST USE | Reason |
|---------------|-------------------|--------------|---------|
| **< 50K LOC** | < 100 issues | **[Standard Strategy](#standard-strategy)** | Fits in memory |
| **50-100K LOC** | 100-200 issues | **[Hybrid Strategy](#hybrid-strategy)** | May exceed limits |
| **> 100K LOC** | > 200 issues | **[Progressive Writing](#progressive-writing-strategy)** | Will overflow |

⚠️ **WARNING**: Choosing wrong strategy will cause:
- Standard on large codebase → 32K token overflow ERROR
- Progressive on small codebase → Unnecessary complexity

### Step 3: Mandatory Reading Order

Based on your chosen strategy, read these documents IN THIS EXACT ORDER:

#### Standard Strategy (<50K LOC)
1. **[COMPLETENESS-ENFORCEMENT.md](COMPLETENESS-ENFORCEMENT.md)** - 3-phase validation system
2. **[CLAUDE-ANALYSIS-FRAMEWORK.md](CLAUDE-ANALYSIS-FRAMEWORK.md)** - Core workflow
3. **[AGENT-PROMPTS.md](AGENT-PROMPTS.md)** - Agent templates
4. **You're done** - Proceed to execution

#### Hybrid Strategy (50-100K LOC)
1. **[UNIVERSAL-CONTEXT-MANAGEMENT.md](UNIVERSAL-CONTEXT-MANAGEMENT.md)** - Memory optimization
2. **[COMPLETENESS-ENFORCEMENT.md](COMPLETENESS-ENFORCEMENT.md)** - Validation system
3. **[CLAUDE-ANALYSIS-FRAMEWORK.md](CLAUDE-ANALYSIS-FRAMEWORK.md)** - Core workflow
4. **[AGENT-PROMPTS.md](AGENT-PROMPTS.md)** - Agent templates

#### Progressive Writing Strategy (>100K LOC)
1. **[UNIVERSAL-CONTEXT-MANAGEMENT.md](UNIVERSAL-CONTEXT-MANAGEMENT.md)** - Section 5.6 specifically
2. **[CLAUDE-ANALYSIS-FRAMEWORK.md](CLAUDE-ANALYSIS-FRAMEWORK.md)** - Progressive workflow
3. **[AGENT-PROMPTS.md](AGENT-PROMPTS.md)** - With write-clear pattern
4. **Initialize output files** before starting:
   ```bash
   touch security_findings.md performance_findings.md
   touch concurrency_findings.md architecture_findings.md
   ```

### Step 4: Execution Patterns by Strategy

#### Standard Strategy
```python
# Pattern: Accumulate in memory, return complete JSON
findings = []
for file in all_files:
    issues = analyze(file)
    findings.extend(issues)
return json.dumps(findings)  # All at once
```

#### Progressive Writing Strategy
```python
# Pattern: Write incrementally, clear memory, return summary only
with open('security_findings.md', 'a') as f:
    findings_batch = []
    for file in all_files:
        issues = analyze(file)
        findings_batch.extend(issues)

        if len(findings_batch) >= 50:  # Every 50 findings
            f.write(format_findings(findings_batch))
            findings_batch = []  # CLEAR MEMORY

    # Write remaining
    if findings_batch:
        f.write(format_findings(findings_batch))

# Return SUMMARY ONLY (not findings!)
return {
    "findings_written": 250,
    "file": "security_findings.md"
}
```

### Step 5: Critical Validation Rules

**ALL strategies must enforce**:

1. **No Summarization**: Never write "Found 8 SQL injections" without listing all 8
2. **Complete Listing**: Every finding must have file:line reference
3. **Validation Block**: Include estimated_range and actual_count check
4. **Progress Reporting**: Report every 10% completion

### AI Model FAQ

**Q: What if I'm unsure about codebase size?**
A: Use Progressive Writing Strategy - it always works and never overflows.

**Q: Can I read all documents regardless of strategy?**
A: Yes, but follow the mandatory order for your chosen strategy.

**Q: What if analysis fails midway?**
A: With Progressive Writing, your findings are already saved to disk. With Standard, you lose everything.

**Q: Should I use multiple agents?**
A: Yes, run Security, Performance, Concurrency, and Architecture agents in parallel when possible.

---

## Part 3: Full Analysis Workflow

For complete workflow scripts and examples, see:
- [SCRIPTS.md](SCRIPTS.md) - All executable scripts
- [EXAMPLES.md](EXAMPLES.md) - Real-world analysis examples

---

## Part 4: Troubleshooting

### Common Issues and Solutions

#### Issue: "Token limit exceeded" error
**Diagnosis**: Codebase too large for chosen strategy
**Solution**:
```bash
# Switch to Progressive Writing Strategy immediately
# Initialize output files
for category in security performance concurrency architecture; do
    echo "# $category Findings" > "${category}_findings.md"
done
# Re-run with progressive writing enabled
```

#### Issue: "Analysis taking too long"
**Diagnosis**: Analyzing too many files without filtering
**Solution**:
```bash
# Use pattern scanning to identify hotspots first
grep -r "critical_pattern" --include="*.java" -l > hotspots.txt
# Analyze only hotspot files
```

#### Issue: "Memory/Context overflow during analysis"
**Diagnosis**: Keeping too much in memory
**Solution**: Use the write-clear pattern
```python
# Wrong: Accumulate everything
all_findings = []
for file in files:
    all_findings.extend(analyze(file))  # Memory grows!

# Right: Write and clear
with open('output.md', 'a') as f:
    batch = []
    for file in files:
        batch.extend(analyze(file))
        if len(batch) >= 50:
            f.write(format_findings(batch))
            batch = []  # Clear memory!
```

---

## Quick Reference

| Action | Command/File |
|--------|-------------|
| Quick scan | `./quick-scan.sh` |
| Full discovery | See [SCRIPTS.md](SCRIPTS.md) |
| Find hotspots | See [SCRIPTS.md](SCRIPTS.md) |
| Progressive setup | `touch {security,performance,concurrency,architecture}_findings.md` |
| Examples | See [EXAMPLES.md](EXAMPLES.md) |
| All scripts | See [SCRIPTS.md](SCRIPTS.md) |

## Document Index

| Document | Purpose | When to Read |
|----------|---------|--------------|
| **START-HERE.md** | This guide - entry point | Always first |
| **COMPLETENESS-ENFORCEMENT.md** | Anti-summarization rules | Before analysis |
| **UNIVERSAL-CONTEXT-MANAGEMENT.md** | Memory optimization | For large codebases |
| **CLAUDE-ANALYSIS-FRAMEWORK.md** | Core methodology | After strategy chosen |
| **AGENT-PROMPTS.md** | Agent templates | When running agents |
| **LANGUAGE-PLUGINS.md** | Language patterns | For specific languages |
| **FRAMEWORK-RULES-HIERARCHY.md** | Conflict resolution | v3.0 - when rules conflict |
| **EXAMPLES.md** | Real-world examples | For reference |
| **SCRIPTS.md** | All executable scripts | For automation |

---

## Version & Support

**Version**: 3.0 (Consolidated Edition)
**Last Updated**: 2025-10-12
**Framework Version**: 3.0

For additional support and patterns, see:
- [EXAMPLES.md](EXAMPLES.md) - Real-world case studies
- [SCRIPTS.md](SCRIPTS.md) - All automation scripts
- [LANGUAGE-PLUGINS.md](LANGUAGE-PLUGINS.md) - Language-specific patterns

**Remember**: When in doubt, use Progressive Writing Strategy - it always works!

---

END OF GUIDE