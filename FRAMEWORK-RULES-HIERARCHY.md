# FRAMEWORK RULES HIERARCHY v3.0

**Version**: 3.0
**Date**: 2025-10-12
**Framework**: claude-code-review-framework
**Purpose**: Define priority order when framework rules conflict

---

## 📚 READING CONTEXT

**This document is referenced from**:
- `UNIVERSAL-CONTEXT-MANAGEMENT.md` (Step 5.5/5.6)
- `CLAUDE-ANALYSIS-FRAMEWORK.md` (Step 3)
- `COMPLETENESS-ENFORCEMENT.md` (Step 5.4)

**Why this document exists**: The framework v2.4 contained logical contradictions between rules. v3.0 introduces a **rule hierarchy** to resolve conflicts.

🎯 **[→ GO TO START-HERE.md](START-HERE.md)** for the complete reading order.

---

## ⚠️ THE PROBLEM (v2.4)

In framework v2.4, these rules conflicted:

### Contradiction Example
```
COMPLETENESS-ENFORCEMENT.md says:
"Document EVERY finding individually - NO summarization"

UNIVERSAL-CONTEXT-MANAGEMENT.md says:
"Progressive Compression: Second occurrence: Store location only"

AGENT-PROMPTS.md says:
"Keep 5 MEDIUM samples detailed (5 lines each)"
```

**Question**: If I find 80 MEDIUM findings, do I:
- A) Document all 80 in detail? (COMPLETENESS)
- B) Compress 75 of them? (CONTEXT MANAGEMENT)
- C) Keep only 5 detailed? (OUTPUT STRATEGY)

**v2.4 had no answer** → Agents couldn't follow all rules simultaneously.

---

## ✅ THE SOLUTION (v3.0)

### Rule Hierarchy

When rules conflict, follow this **priority order**:

```
┌─────────────────────────────────────────────────┐
│  PRIORITY 1: COMPLETENESS (Non-Negotiable)     │
│  "You MUST find and document ALL findings"     │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│  PRIORITY 2: CONTEXT MANAGEMENT (Enabler)      │
│  "HOW you achieve completeness within limits"  │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│  PRIORITY 3: OUTPUT STRATEGY (Presentation)    │
│  "HOW you present findings to the user"        │
└─────────────────────────────────────────────────┘
```

---

## 📋 PRIORITY 1: COMPLETENESS

**Source**: `COMPLETENESS-ENFORCEMENT.md`

### Core Rules (Cannot be violated)

1. **Pre-Analysis ESTIMATION** (v3.0)
   - Declare confidence interval `[min, max]` before analyzing
   - Example: "Estimated 36-77 findings (±35% confidence)"
   - ✅ Realistic: You cannot know exact count before analyzing

2. **Progressive Extraction**
   - Analyze 100% of codebase (or specified sampling %)
   - Find ALL instances of each pattern
   - Report progress every 10%

3. **Output Validation** (v3.0)
   - Verify `actual_count` within `[min_estimate, max_estimate]`
   - If outside range: document reason
   - ✅ Range-based validation (not exact match)

### What COMPLETENESS Means

- **During Analysis**: Find EVERY finding (100% discovery)
- **During Storage**: Write ALL findings to disk
- **During Presentation**: May sample for final output (see Priority 3)

**Key Insight**: Completeness is about **FINDING all issues**, not necessarily **PRESENTING all issues in full detail**.

---

## 🔧 PRIORITY 2: CONTEXT MANAGEMENT

**Source**: `UNIVERSAL-CONTEXT-MANAGEMENT.md`

### Core Rules (Techniques to enable Priority 1)

Context Management describes **HOW** to achieve completeness within token limits:

#### Technique 1: Progressive Compression (IN-MEMORY)
```
During analysis (in agent's working memory):
- First occurrence: Store full details
- Second occurrence: Store location only
- Third+ occurrence: Increment counter

Purpose: Reduce memory footprint while analyzing
```

#### Technique 2: Progressive Writing (TO DISK)
```
During analysis (write to disk incrementally):
- Every 50 findings (or dynamic): Flush to disk
- Clear from memory after writing
- Continue with freed context

Purpose: Prevent context overflow for large codebases
```

#### Technique 3: Dynamic Write Intervals (v3.0)
```
Adjust write frequency based on context usage:
- Context < 70%: write every 50 findings
- Context 70-85%: write every 25 findings
- Context 85-95%: write every 10 findings
- Context > 95%: write immediately (every 1 finding)

Purpose: Adaptive context management
```

#### Technique 4: Strategic Sampling (>500K LOC only)
```
For extremely large codebases (>500K LOC):
- Analyze minimum 40% strategically
- 100% of entry points
- 80% of business logic
- 40% of data layer
- 100% of security-critical paths

Purpose: Make analysis tractable for massive codebases
```

### What CONTEXT MANAGEMENT Means

- **Compression is temporary** (during analysis only)
- **All findings written to disk** (no loss)
- **Final output expands** (see Priority 3)

**Key Insight**: Context management is a **TECHNIQUE** to achieve completeness, not a reason to skip findings.

---

## 📊 PRIORITY 3: OUTPUT STRATEGY

**Source**: `AGENT-PROMPTS.md`, `UNIVERSAL-CONTEXT-MANAGEMENT.md`

### Core Rules (How to present findings to user)

Output Strategy describes **HOW** to present the findings that were found and stored:

#### v3.0 Unified Strategy (Count-Based)

**CRITICAL findings**:
- Rule: Keep ALL in detailed format
- Reason: Immediate security/data corruption risks
- Never sampled, never compressed in final output

**HIGH findings**:
- Rule: Keep ALL in detailed format
- Reason: Significant impact on system
- Never sampled, never compressed in final output

**MEDIUM findings** (count-based):
```
If < 20 total: Keep ALL detailed
If 20-50 total: Keep top 10 detailed + Quick Reference Table
If > 50 total: Keep top 5 detailed + Quick Reference Table

Reason: Balance detail vs context usage
```

**LOW findings** (count-based):
```
If < 15 total: Keep ALL detailed
If 15-40 total: Keep top 8 detailed + Quick Reference Table
If > 40 total: Keep top 3 detailed + Quick Reference Table

Reason: Provide representative samples + full index
```

**Quick Reference Table**:
```markdown
| ID | Severity | Category | File:Line | Brief Description |
|----|----------|----------|-----------|-------------------|
| SEC-051 | MEDIUM | INPUT | Auth.java:45 | Missing @Valid |
```

### What OUTPUT STRATEGY Means

- **All findings are documented** (100% in Quick Ref Table)
- **Some findings detailed** (based on count rules)
- **User sees ALL findings** (detailed or in table)

**Key Insight**: Output strategy is about **PRESENTATION FORMAT**, not about skipping findings.

---

## 🎯 CONFLICT RESOLUTION EXAMPLES

### Example 1: 80 MEDIUM Findings

**Conflict**:
- COMPLETENESS says: "Document EVERY finding"
- CONTEXT MANAGEMENT says: "Compress to save memory"
- OUTPUT STRATEGY says: "Keep top 5 detailed (>50 rule)"

**Resolution** (using hierarchy):

1. **Priority 1 (COMPLETENESS)**:
   - ✅ Analyze ALL code and find all 80 MEDIUM findings
   - ✅ Write all 80 to disk via Progressive Writing

2. **Priority 2 (CONTEXT MANAGEMENT)**:
   - ✅ Use progressive compression in memory (first=full, rest=location)
   - ✅ Write every 50 findings to disk, clear memory
   - ✅ All 80 stored on disk

3. **Priority 3 (OUTPUT STRATEGY)**:
   - ✅ Final output: Top 5 detailed + Quick Reference Table (75 entries)
   - ✅ User sees ALL 80 findings (5 detailed, 75 in table)

**Result**: No conflict. All rules satisfied in their priority order.

### Example 2: 18 MEDIUM Findings

**Conflict**:
- OUTPUT STRATEGY says: "<20 = keep ALL detailed"
- CONTEXT MANAGEMENT says: "Compress similar findings"

**Resolution**:

1. **Priority 1 (COMPLETENESS)**:
   - ✅ Find all 18 MEDIUM findings

2. **Priority 2 (CONTEXT MANAGEMENT)**:
   - ✅ Compress in memory during analysis
   - ✅ Write all 18 to disk

3. **Priority 3 (OUTPUT STRATEGY)**:
   - ✅ Count rule: <20 = ALL detailed
   - ✅ Final output: All 18 in detailed format
   - ✅ No Quick Reference Table needed

**Result**: When count is low, all findings get detailed format.

### Example 3: Pre-Analysis Estimation

**Conflict (v2.4)**:
- COMPLETENESS said: "Declare exact count BEFORE analyzing"
- Reality: Impossible to know exact count before analyzing

**Resolution (v3.0)**:

1. **Priority 1 (COMPLETENESS v3.0)**:
   - ✅ Declare confidence interval: [36-77 findings, ±35%]
   - ✅ Validation: actual_count within range OR document variance
   - ❌ OLD (v2.4): declared_count === actual_count (impossible)

**Result**: v3.0 uses realistic estimation, not impossible exact count.

### Example 4: Strategic Sampling (>500K LOC)

**Conflict**:
- COMPLETENESS says: "Analyze 100%"
- Codebase size: 800K LOC (would overflow context)

**Resolution**:

1. **Priority 1 (COMPLETENESS)**:
   - ✅ Acknowledge sampling is required for >500K LOC
   - ✅ Document sampling strategy and confidence

2. **Priority 2 (CONTEXT MANAGEMENT)**:
   - ✅ Analyze minimum 40% strategically:
     - 100% entry points
     - 80% business logic
     - 40% data layer
     - 100% security paths
   - ✅ Write all findings to disk

3. **Priority 3 (OUTPUT STRATEGY)**:
   - ✅ Apply count-based rules to findings found
   - ✅ Report: "Analysis coverage: 45% (strategic sampling)"

**Result**: Completeness adapts to reality (sampling documented), not violated.

---

## 🚫 ANTI-PATTERNS (What NOT to do)

### ❌ Anti-Pattern 1: Violating Priority 1
```
BAD: "I found 80 MEDIUM findings but only analyzed 10 because
      the output strategy says keep 5 samples"

GOOD: "I found 80 MEDIUM findings by analyzing ALL code.
       Wrote all 80 to disk. Final output has 5 detailed + 75 in table."
```

### ❌ Anti-Pattern 2: Confusing Compression with Skipping
```
BAD: "Progressive compression means I only document the first
      occurrence and skip the rest"

GOOD: "Progressive compression means I compress IN MEMORY during
       analysis, but write ALL findings to disk"
```

### ❌ Anti-Pattern 3: Exact Count Declaration (v2.4)
```
BAD: "I declare there are exactly 52 findings before analyzing"
     (impossible to know)

GOOD: "I estimate 36-77 findings (±35% confidence) based on
       preliminary grep scan"
```

### ❌ Anti-Pattern 4: Sampling Without Justification
```
BAD: "Codebase is 120K LOC, so I'll sample 20% to save time"

GOOD: "Codebase is 120K LOC. Using Progressive Writing Strategy
       to analyze 100% without context overflow"
```

---

## ✅ VALIDATION CHECKLIST

Use this checklist to verify you're following the hierarchy:

### During Analysis
- [ ] **Priority 1**: Did I analyze 100% of codebase (or documented sampling %)?
- [ ] **Priority 1**: Did I find ALL instances of each pattern?
- [ ] **Priority 2**: Did I use progressive compression to manage memory?
- [ ] **Priority 2**: Did I write findings to disk progressively?
- [ ] **Priority 2**: Did I clear memory after each write?

### During Output Generation
- [ ] **Priority 1**: Are ALL findings documented (detailed or in Quick Ref)?
- [ ] **Priority 3**: Did I apply count-based rules for MEDIUM/LOW?
- [ ] **Priority 3**: Did I keep ALL CRITICAL/HIGH detailed?
- [ ] **Priority 3**: Did I create Quick Reference Table if needed?

### During Validation
- [ ] **Priority 1**: Is actual_count within [min_estimate, max_estimate]?
- [ ] **Priority 1**: If outside range, did I document the reason?
- [ ] **Priority 3**: Does breakdown match count-based rules?

---

## 📖 INTEGRATION WITH OTHER DOCUMENTS

### How This Document Relates to Framework

```
START-HERE.md (Reading Order)
    ↓
CLAUDE-ANALYSIS-FRAMEWORK.md (Overall Workflow)
    ↓
AGENT-PROMPTS.md (Agent Templates)
    ↓
┌───────────────────────────────────────────┐
│  When executing agents, you need...      │
│                                           │
│  COMPLETENESS-ENFORCEMENT.md ←──┐        │
│  UNIVERSAL-CONTEXT-MANAGEMENT.md │        │
│  FRAMEWORK-RULES-HIERARCHY.md ←──┘        │
│         (this document)                   │
│                                           │
│  Read all 3 to understand:                │
│  - WHAT to achieve (completeness)         │
│  - HOW to achieve it (context mgmt)       │
│  - PRIORITY when conflicts arise (hier.)  │
└───────────────────────────────────────────┘
    ↓
LANGUAGE-PLUGINS.md (Language-Specific Patterns)
```

### When to Reference This Document

**Read this document when**:
- You encounter conflicting instructions
- You're unsure whether to compress findings
- You need to decide between detailed vs table format
- You're validating actual_count vs estimates

**This document answers**:
- "Do I analyze 100% or can I sample?"
- "Do I document all findings or just top N?"
- "Can I compress CRITICAL findings?"
- "What if I find more findings than estimated?"

---

## 🎓 KEY TAKEAWAYS

1. **Hierarchy exists to RESOLVE conflicts**, not create restrictions
2. **Completeness is about FINDING all**, not PRESENTING all in full detail
3. **Context Management is a TECHNIQUE**, not an excuse to skip analysis
4. **Output Strategy is PRESENTATION**, not discovery
5. **v3.0 uses estimation ranges**, not impossible exact counts
6. **All findings are documented** (detailed or in Quick Reference Table)

---

## 📚 SEE ALSO

- `START-HERE.md` - Framework reading order and decision points
- `COMPLETENESS-ENFORCEMENT.md` - Full specification of completeness rules
- `UNIVERSAL-CONTEXT-MANAGEMENT.md` - Context management techniques
- `AGENT-PROMPTS.md` - Agent templates with integrated v3.0 rules
- `CLAUDE-ANALYSIS-FRAMEWORK.md` - Overall 6-phase workflow

---

## VERSION HISTORY

**v3.0 (2025-10-12)**:
- Initial creation of FRAMEWORK-RULES-HIERARCHY.md
- Resolves 7 logical contradictions from v2.4
- Defines 3-level priority hierarchy
- Provides conflict resolution examples

---

**Remember**: The hierarchy exists to make the framework **usable**, not restrictive. When in doubt, prioritize **finding all issues** (Priority 1) over **presentation concerns** (Priority 3).
