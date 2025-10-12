# START HERE - AI Model Reading Guide
## Claude Code Review Framework

**Version**: 2.3
**Last Updated**: 2025-10-12
**Audience**: AI Models (Claude, GPT, etc.) tasked with code review

---

## 🎯 PURPOSE

This guide tells you **HOW TO READ** the framework documentation in the correct order to perform comprehensive code reviews.

---

## ⚠️ BEFORE YOU START - DETERMINE YOUR STRATEGY

**MANDATORY ASSESSMENT** before reading further. Answer these questions honestly:

### 1. Codebase Size Assessment
Check ONE option that matches your target codebase:

- [ ] **< 50K LOC** → Read Steps 1-4, use Standard Output
- [ ] **50-100K LOC** → Read Steps 1-5.5, consider Progressive Writing
- [ ] **> 100K LOC** → Read Steps 1-5.6 **MANDATORY**, use Progressive Writing

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

### Step 2: Understand Completeness Enforcement
**File**: `COMPLETENESS-ENFORCEMENT.md`
**Time**: 5 minutes
**Purpose**: Learn the **THREE-PHASE** process that prevents you from summarizing findings

**Why Critical**: Without reading this, you WILL make the mistake of writing:
- ❌ "Found 8 SQL injection vulnerabilities"
- ✅ Instead of listing all 8 individually with file:line

**Key Concepts**:
- Phase 1: Pre-Analysis Counting
- Phase 2: Progressive Extraction (10% checkpoints)
- Phase 3: Output Validation

**After reading, you MUST**:
- Always declare expected finding count BEFORE analyzing
- Report progress every 10%
- Output JSON with `analysis_metadata` and `validation` blocks

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

### Step 5 (Optional): Quick Start Examples
**File**: `QUICK-START.md`
**Time**: 10 minutes
**Purpose**: See real-world examples

**Contains**:
- 5-minute quick start workflow
- 30-minute full analysis workflow
- Examples for Spring Boot, Django, Node.js
- Troubleshooting common issues

---

### Step 5.5 (CRITICAL - NEW v2.2): Universal Context Management
**File**: `UNIVERSAL-CONTEXT-MANAGEMENT.md`
**Time**: 8 minutes
**Purpose**: Master smart compression for large codebases
**READ BEFORE analyzing codebases >50K LOC**

**Why Critical**: Context window is limited (200K tokens). Large projects need intelligent compression.

**Key Principles**:
1. **Monitor context actively** (check every 10 files)
2. **Equal domain priority** (25% Security, 25% Performance, 25% Concurrency, 25% Architecture)
3. **Progressive compression** (full details → compressed → pattern codes)
4. **Smart sampling** (for >100K LOC codebases)

**Context Thresholds**:
```
0-60%: Full analysis
60-70%: Start batching similar findings
70-80%: CRITICAL + HIGH only
80-90%: CRITICAL only with pattern codes
>90%: Emergency output
```

**Adaptive Strategies**:
- **<10K LOC**: Single pass, full details
- **10-50K LOC**: Layer-based chunks
- **50-100K LOC**: Pattern-based chunks with compression
- **>100K LOC**: Strategic sampling (60% coverage)

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

### Step 5.6 (⚠️ MANDATORY for >100K LOC - NEW v2.3): Progressive Writing Strategy
**File**: `UNIVERSAL-CONTEXT-MANAGEMENT.md` (Section: Progressive Writing v2.3)
**Time**: 5 minutes
**Purpose**: Solve the 32K output token limit for large-scale analysis
**⚠️ READ BEFORE analyzing codebases with >100K LOC or 100+ expected findings**

**Why Critical**: Claude has a 32,000 token OUTPUT limit (~40KB). Large analyses generate 800+ findings = 50KB+ output → OVERFLOW ERROR.

**The Problem**:
```
Agent finds 250 issues → tries to return all in JSON → exceeds 32K token limit → ERROR
```

**The Solution - Write-Clear-Continue Pattern**:
```bash
# Don't accumulate findings in memory, write incrementally to disk

findings_count=0
BATCH_SIZE=50

for file in $(find . -name "*.java" | sort); do
    findings=$(analyze_security "$file")

    for finding in $findings; do
        # Write to output file immediately
        echo "$finding" >> security_findings.md
        findings_count=$((findings_count + 1))

        # Every 50 findings, CLEAR from context
        if [ $((findings_count % BATCH_SIZE)) -eq 0 ]; then
            echo "[Progress] $findings_count findings written to disk"
            # Context freed - can continue with constant memory
        fi
    done
done
```

**Agent Architecture**:
- Each agent writes to **separate category file**:
  - Security Agent → `security_findings.md`
  - Performance Agent → `performance_findings.md`
  - Concurrency Agent → `concurrency_findings.md`
  - Architecture Agent → `architecture_findings.md`

**Agent Returns Summary Only** (NOT full findings):
```json
{
  "agent": "Security Agent",
  "status": "completed",
  "output_file": "security_findings.md",
  "findings_found": 250,
  "findings_documented": 102,
  "sampling_applied": true,
  "sampling_strategy": {
    "CRITICAL": "ALL (50 documented)",
    "HIGH": "ALL (32 documented)",
    "MEDIUM": "30% sampled (15 of 50 documented)",
    "LOW": "20% sampled (5 of 25 documented)"
  }
}
```

**Key Benefits**:
- ✅ **Constant Memory**: Write → Clear → Context stays at ~50KB regardless of findings
- ✅ **No Output Overflow**: Return 2KB summary instead of 40KB+ findings
- ✅ **Scales to 1M+ LOC**: Write 10,000 findings without hitting limits
- ✅ **67% Context Savings**: Proven on 138K LOC project (362+ findings)

**Intelligent Sampling**:
- **CRITICAL**: Document ALL (no omissions)
- **HIGH**: Document ALL (no omissions)
- **MEDIUM**: Sample top 30% by impact
- **LOW**: Sample top 20% by frequency

**When to Use**:
- Codebase >100K LOC
- Expected findings >100 issues
- Multiple parallel agents
- Deep comprehensive analysis

**Read Full Section**: `UNIVERSAL-CONTEXT-MANAGEMENT.md` lines 50-150 for:
- Complete implementation with bash examples
- Incremental writing patterns
- Sampling algorithms
- Context monitoring during writes

---

### Step 6: Choose Your Execution Strategy (MANDATORY DECISION POINT)

**Based on your assessment from "BEFORE YOU START" section**, select the appropriate execution strategy:

#### Strategy A: Standard Output (Small/Medium Codebases)

**✅ Use when**:
- Codebase < 100K LOC **AND**
- Expected findings < 100 issues

**How it works**:
- Agents analyze code and accumulate findings in memory
- Return complete JSON response with all findings
- Validate: `declared_count === actual_count`
- Maximum output: ~30KB (safe within 32K token limit)

**Agent response format**:
```json
{
  "analysis_metadata": { ... },
  "findings": [ ... array of all findings ... ],
  "validation": { "counts_match": true }
}
```

**Pros**: Simple, all findings in single response
**Cons**: Fails with 32K overflow if too many findings

---

#### Strategy B: Progressive Writing (Large Codebases) - v2.3

**✅ Use when**:
- Codebase > 100K LOC **OR**
- Expected findings > 100 issues **OR**
- When unsure (safest choice)

**How it works**:
1. Each agent writes to **separate category file** during analysis:
   - Security Agent → `security_findings.md`
   - Performance Agent → `performance_findings.md`
   - Concurrency Agent → `concurrency_findings.md`
   - Architecture Agent → `architecture_findings.md`

2. **Write-Clear-Continue pattern**:
   ```bash
   for each file in codebase:
       analyze and identify findings
       write findings to disk immediately
       if findings_count % 50 == 0:
           flush to disk
           CLEAR findings from context  # Critical!
           continue analysis
   ```

3. Apply intelligent sampling:
   - CRITICAL: Document ALL (no omissions)
   - HIGH: Document ALL (no omissions)
   - MEDIUM: Sample top 30% by impact
   - LOW: Sample top 20% by frequency

4. Agent returns **summary only** (2KB instead of 40KB+):
   ```json
   {
     "agent": "Security Agent",
     "status": "completed",
     "output_file": "security_findings.md",
     "findings_found": 250,
     "findings_documented": 102,
     "sampling_applied": true
   }
   ```

**Pros**: Never exceeds token limits, scales to 1M+ LOC
**Cons**: Findings split across multiple files (minor inconvenience)

---

### ⚠️ Strategy Selection Consequences

| Your Choice | What Happens |
|-------------|--------------|
| Standard output with 200+ findings | **32,000 TOKEN OVERFLOW ERROR** 💥 Analysis fails completely |
| Standard output with 80 findings | ✅ Works perfectly, all findings in response |
| Progressive Writing with 200+ findings | ✅ Works perfectly, findings in files |
| Progressive Writing with 80 findings | ✅ Works (unnecessary overhead but safe) |

### 🎯 Decision Algorithm

```
IF (codebase_size > 100K OR expected_findings > 100):
    strategy = PROGRESSIVE_WRITING  # MANDATORY
    initialize_output_files()
    use_write_clear_continue_pattern()
ELSE:
    strategy = STANDARD_OUTPUT
    accumulate_findings_in_memory()
    return_full_json()
```

**Golden Rule**: **When in doubt, use Progressive Writing** (always works, never fails)

---

## 🚀 EXECUTION WORKFLOW

When you receive a code review request:

### **Before Starting Analysis**

```
1. Read project code structure
2. Create manifest.json (Phase 1)
3. Run pattern scans with grep (Phase 2)
4. Load COMPLETENESS-ENFORCEMENT.md into context
5. Load appropriate agent templates from AGENT-PROMPTS.md
```

### **During Analysis (For EACH Agent)**

```
✅ PHASE 1: PRE-ANALYSIS COUNTING
   - Count expected findings by category
   - Declare total count
   - Output pre_analysis_count JSON

✅ PHASE 2: PROGRESSIVE EXTRACTION (with Progressive Writing v2.3)
   - Initialize output file: security_findings.md (or category-specific file)
   - For each finding:
     * Analyze and document
     * Write to file immediately
     * Increment counter
   - Every 50 findings:
     * Flush to disk
     * CLEAR findings from context
     * Report progress: [Progress] X findings written to disk
   - Continue with freed context

   Progress Checkpoints:
   [10%] X/Total findings extracted (written to file)
   [20%] X/Total findings extracted (written to file)
   ...
   [100%] Total/Total findings extracted ✓

✅ PHASE 2.5: INTELLIGENT SAMPLING (for large finding sets)
   - Apply sampling strategy:
     * CRITICAL: Keep ALL
     * HIGH: Keep ALL
     * MEDIUM: Keep top 30% by impact
     * LOW: Keep top 20% by frequency
   - Document sampling metadata in output

✅ PHASE 3: OUTPUT VALIDATION
   - Generate analysis_metadata with sampling info
   - Return SUMMARY ONLY (not full findings - already written to file)
   - Generate validation block
   - Verify: findings_found, findings_documented, sampling_applied
```

### **After Analysis**

```
1. Validate output with Phase 5.5 script
2. If validation fails → re-run agent
3. If validation passes → merge findings
4. Deduplicate across agents
5. Generate final report
```

---

## ⚠️ CRITICAL RULES (DO NOT SKIP)

### Rule #1: NO SUMMARIZATION
```
❌ WRONG:
"Found 8 SQL injection vulnerabilities across repository files"

✅ CORRECT:
SEC-001: SQL injection in UserRepository.java:45
SEC-002: SQL injection in OrderRepository.java:89
SEC-003: SQL injection in PaymentRepository.java:123
... (all 8 listed individually)
```

### Rule #2: ALWAYS COUNT FIRST
Before analyzing ANY code:
```json
{
  "pre_analysis_count": {
    "declared_finding_count": 52,
    "files_to_analyze": 30,
    "categories": {
      "SQL_INJECTION": 15,
      "MISSING_AUTH": 12,
      "HARDCODED_SECRETS": 5,
      "INPUT_VALIDATION": 20
    }
  }
}
```

### Rule #3: PROGRESS TRACKING
Report every 10% with specific finding IDs:
```
[10%] 5/52 findings extracted
  ├─ SEC-001: SQL injection in UserRepository.java:45
  ├─ SEC-002: SQL injection in OrderRepository.java:89
  ├─ SEC-003: SQL injection in PaymentRepository.java:123
  ├─ SEC-004: Missing auth in AdminController.java:34
  └─ SEC-005: Missing auth in UserController.java:67
```

### Rule #4: VALIDATION BLOCK MANDATORY
Every agent output MUST include:
```json
{
  "validation": {
    "id_sequence_valid": true,
    "no_duplicates": true,
    "all_have_evidence": true,
    "all_have_recommendations": true,
    "counts_match": true
  }
}
```

If ANY validation is `false`, output is REJECTED.

---

## 📁 FILE REFERENCE

### Core Framework Files
| File | Purpose | Read When |
|------|---------|-----------|
| `START-HERE.md` | Reading guide | FIRST |
| `COMPLETENESS-ENFORCEMENT.md` | Anti-summarization rules | SECOND (CRITICAL) |
| `CLAUDE-ANALYSIS-FRAMEWORK.md` | Overall workflow | THIRD |
| `AGENT-PROMPTS.md` | Agent templates | FOURTH |
| `QUICK-START.md` | Examples | OPTIONAL |
| `UNIVERSAL-CONTEXT-MANAGEMENT.md` | Context optimization | CRITICAL for >50K LOC |

### When Executing Code Review

**For ALL projects**:
1. Load `COMPLETENESS-ENFORCEMENT.md` → internalize rules
2. Load `AGENT-PROMPTS.md` → copy relevant agent template
3. Reference `CLAUDE-ANALYSIS-FRAMEWORK.md` → for workflow phases

**For large codebases (>50K LOC)**:
4. Load `UNIVERSAL-CONTEXT-MANAGEMENT.md` → apply compression strategy
5. Monitor context usage every 10 files
6. Apply adaptive chunking based on size

---

## 🎓 LEARNING CHECKLIST

Before starting your first code review, verify you understand:

### Completeness Enforcement
- [ ] What is the three-phase process?
- [ ] Why is pre-analysis counting mandatory?
- [ ] What triggers output rejection?
- [ ] How to avoid summarization?

### Agent Execution
- [ ] What is the output JSON schema?
- [ ] What fields are mandatory in each finding?
- [ ] How to report progress during extraction?
- [ ] What goes in `analysis_metadata` vs `findings` vs `validation`?

### Framework Workflow
- [ ] What is semantic segmentation?
- [ ] How to use pattern scanning (grep) before deep analysis?
- [ ] What is the token budget per agent?
- [ ] When to split large layers into sub-agents?

---

## 💡 QUICK REFERENCE

### If You're Analyzing...

**Java Spring Boot Project**:
```
Read Order: START-HERE → COMPLETENESS-ENFORCEMENT →
            AGENT-PROMPTS (JPA, Performance, Resilience, Security)
Focus: Hibernate entities, Feign clients, @Transactional boundaries
```

**Python Django Project**:
```
Read Order: START-HERE → COMPLETENESS-ENFORCEMENT →
            AGENT-PROMPTS (Security, Performance, ORM sections)
Focus: ORM N+1, select_related, authentication, SQL injection
```

**JavaScript Node.js Project**:
```
Read Order: START-HERE → COMPLETENESS-ENFORCEMENT →
            AGENT-PROMPTS (Security, Performance, Concurrency)
Focus: XSS, event loop blocking, async/await, prototype pollution
```

---

## 🔄 WORKFLOW DIAGRAM

```
┌─────────────────────────────────────────────────────────────┐
│  AI Model Receives Code Review Request                      │
└─────────────────┬───────────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────────┐
│  Step 1: Read START-HERE.md (this file)                     │
│  → Understand reading order                                  │
└─────────────────┬───────────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────────┐
│  Step 2: Read COMPLETENESS-ENFORCEMENT.md                   │
│  → Internalize 3-phase process                              │
│  → Study anti-summarization examples                         │
└─────────────────┬───────────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────────┐
│  Step 3: Read CLAUDE-ANALYSIS-FRAMEWORK.md                  │
│  → Understand Phase 0-6 workflow                             │
│  → Learn validation requirements (Phase 5.5)                 │
└─────────────────┬───────────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────────┐
│  Step 4: Read AGENT-PROMPTS.md                              │
│  → Load agent templates for detected language                │
│  → Study output schema and examples                          │
└─────────────────┬───────────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────────┐
│  BEGIN ANALYSIS                                              │
│  ├─ Phase 1: PRE-ANALYSIS COUNTING                          │
│  ├─ Phase 2: PROGRESSIVE EXTRACTION (10% checkpoints)       │
│  └─ Phase 3: OUTPUT VALIDATION                              │
└─────────────────┬───────────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────────┐
│  OUTPUT VALIDATION (orchestrator runs validation script)     │
│  ├─ Check: declared_count === actual_count                   │
│  ├─ Check: No summarization keywords                         │
│  ├─ Check: All required fields present                       │
│  └─ If PASS → Accept / If FAIL → Reject & Re-run            │
└─────────────────────────────────────────────────────────────┘
```

---

## ❓ FAQ FOR AI MODELS

### Q: I found 50 SQL injection vulnerabilities. Can I summarize them?
**A**: NO. List all 50 individually with file:line evidence. Use progress tracking:
```
[10%] 5/50 findings extracted
[20%] 10/50 findings extracted
...
[100%] 50/50 findings extracted ✓
```

### Q: What if I estimate wrong in pre-analysis counting?
**A**: Update the count and explain:
```json
{
  "pre_analysis_count": {
    "declared_finding_count": 30,
    "updated_count": 35,
    "reason": "Found 5 additional hardcoded secrets in config files"
  }
}
```

### Q: Can I skip validation block if all checks pass?
**A**: NO. Validation block is MANDATORY in every output.

### Q: What if analysis would exceed token budget?
**A**: **USE PROGRESSIVE WRITING STRATEGY** (Step 5.6) - This is THE solution for token limits:

**Primary solution** (v2.3):
1. **Progressive Writing Strategy**:
   - Write findings to disk DURING analysis (not at end)
   - Flush every 50 findings → clear from context
   - Return summary only (2KB instead of 40KB+)
   - Never exceeds 32K output token limit
   - See Step 5.6 for complete implementation

**Legacy alternatives** (only if Progressive Writing unavailable):
1. Split layer into smaller batches
2. Run sequentially
3. Maintain count across batches
4. Merge at end

**Note**: The legacy approach is a workaround. Progressive Writing is the designed solution for large-scale analysis.

### Q: How detailed should code evidence be?
**A**: Max 10 lines of actual code from the file. Include enough context to understand the issue.

---

## 🎯 SUCCESS CRITERIA

Your output is COMPLETE when:

### For All Analyses (Standard + Progressive Writing):
✅ All findings have file:line references
✅ All findings have code evidence
✅ No placeholder text ("...", "etc.", "and others")
✅ No summarization statements
✅ ID sequence is continuous (no gaps)

### For Standard Output (<100 findings):
✅ `declared_count === actual_count`
✅ Validation block present with all `true` values
✅ Complete JSON with all findings in response

### For Progressive Writing (>100 findings or >100K LOC):
✅ **Separate output files created** per agent category:
   - `security_findings.md` exists with findings
   - `performance_findings.md` exists with findings
   - `concurrency_findings.md` exists with findings
   - `architecture_findings.md` exists with findings
✅ **Write-clear-continue pattern used** (flushed every 50 findings)
✅ **Returned summary only** (NOT full findings in response)
✅ **Summary includes**:
   - `findings_found`: total count discovered
   - `findings_documented`: actual count written to file
   - `output_file`: filename where findings are stored
   - `sampling_applied`: true/false
✅ **Intelligent sampling applied** correctly:
   - CRITICAL: ALL documented (no omissions)
   - HIGH: ALL documented (no omissions)
   - MEDIUM: Top 30% by impact documented
   - LOW: Top 20% by frequency documented

---

## 🚨 COMMON MISTAKES TO AVOID

| ❌ Mistake | ✅ Correct |
|-----------|-----------|
| "Found 8 SQL injections" | List all 8 with file:line |
| Skipping pre-analysis count | Always count before analyzing |
| No progress updates | Report every 10% |
| Missing validation block | Always include validation |
| Guessing technologies | Only report detected frameworks |
| Invented information | If unclear, state "Not determinable" |

---

## 📚 ADDITIONAL RESOURCES

- **Language Plugins**: See `CLAUDE-ANALYSIS-FRAMEWORK.md` sections 794-870
- **Token Budget Management**: See `CLAUDE-ANALYSIS-FRAMEWORK.md` lines 1088-1113
- **Deduplication Algorithm**: See `CLAUDE-ANALYSIS-FRAMEWORK.md` lines 462-498
- **Real Examples**: See `QUICK-START.md` for Spring Boot, Django, Node.js

---

## 📝 VERSION HISTORY

- **v2.3** (2025-10-12): Progressive Writing Strategy - incremental disk writes to bypass 32K output limit
- **v2.2** (2025-10-12): Universal Context Management with adaptive compression strategies
- **v2.1** (2025-10-11): Added completeness enforcement, START-HERE guide
- **v2.0** (2025-10-10): Multi-agent framework with language plugins
- **v1.0** (2025-10-09): Initial framework

---

**Remember**: The goal is **100% completeness**. Document EVERY finding, no matter how many. The framework's three-phase validation ensures you cannot accidentally omit findings.

**Now proceed to**: `COMPLETENESS-ENFORCEMENT.md`
