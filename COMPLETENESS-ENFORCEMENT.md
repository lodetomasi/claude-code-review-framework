# COMPLETENESS ENFORCEMENT - AI EXECUTION RULES

**Version**: 3.0 (AI-Optimized)
**Mandatory Reading**: Step 2 in execution sequence

---

## ⛔ ABSOLUTE PROHIBITIONS - ANTI-SUMMARIZATION

**VIOLATION = ANALYSIS REJECTED - OUTPUT INVALID**

1. ❌ **FORBIDDEN** to write "Found N issues" without listing all N individually
2. ❌ **FORBIDDEN** to group similar findings (e.g., "8 SQL injections across files")
3. ❌ **FORBIDDEN** to use placeholders: "...", "etc.", "and others", "similar issues in X files"
4. ❌ **FORBIDDEN** to omit findings because "they are repetitive"
5. ❌ **FORBIDDEN** to produce output where `actual_count` ≠ number of findings listed
6. ❌ **FORBIDDEN** to have ID sequence gaps (e.g., SEC-001, SEC-003 missing SEC-002)
7. ❌ **FORBIDDEN** to skip Phase 1 estimation before analysis
8. ❌ **FORBIDDEN** to skip progress tracking during extraction
9. ❌ **FORBIDDEN** to skip validation block in final output
10. ❌ **FORBIDDEN** to return findings without file:line:code_snippet

---

## 🚨 FATAL ERRORS - COMPLETENESS VIOLATIONS

### FATAL-101: Summarization Detected
- **Condition**: Output contains phrases like "Found 8 issues" or "Multiple patterns detected"
- **Consequence**: Analysis INVALID - output REJECTED
- **Recovery**: List ALL findings individually with unique IDs

### FATAL-102: Count Mismatch
- **Condition**: `actual_count` outside `[min_estimate, max_estimate]` without documented variance
- **Consequence**: Analysis INCOMPLETE - output REJECTED
- **Recovery**: Re-analyze and document all findings OR explain variance

### FATAL-103: ID Sequence Gaps
- **Condition**: Missing IDs in sequence (e.g., SEC-001, SEC-003 exist but SEC-002 missing)
- **Consequence**: Findings INCOMPLETE - output REJECTED
- **Recovery**: Fill gaps or renumber sequence

### FATAL-104: Missing Required Fields
- **Condition**: Any finding lacks: id, file, line, code_snippet, description, recommendation
- **Consequence**: Finding INVALID - cannot be actioned
- **Recovery**: Complete all required fields for ALL findings

---

## THREE-PHASE MANDATORY PROCESS

You MUST follow this process:

---

### PHASE 1: PRE-ANALYSIS ESTIMATION (MANDATORY)

**Step 1**: Run preliminary grep/pattern scan

**Step 2**: Complete estimation table:

| Finding Category | Estimate Range | Confidence |
|------------------|----------------|------------|
| [Category]       | min-max        | ±X%        |
| **TOTAL**        | **min-max**    | **±X%**    |

**Output Format**:
```json
{
  "pre_analysis_estimation": {
    "estimated_range": {"min": X, "max": Y},
    "confidence_level": "±Z%",
    "files_to_analyze": N,
    "categories": { "CATEGORY": {"min": A, "max": B} },
    "estimation_method": "grep scan + extrapolation"
  }
}
```

---

### PHASE 2: EXTRACTION WITH PROGRESS TRACKING (MANDATORY)

**Report progress every 10%:**

```
[10%] X/N findings extracted
[20%] X/N findings extracted
...
[100%] N/N findings extracted ✓ COMPLETE
```

**Rules**:
- Progress report every 10% OR every 10 findings
- List finding IDs in each batch
- Final count MUST be within [min_estimate, max_estimate]
- If outside range: document variance_reason

**Progressive Writing (Context Management)**:

See **[SAMPLING-RULES.md](SAMPLING-RULES.md#-dynamic-write-intervals-v30)** for Dynamic Write Interval specification.

---

### PHASE 3: OUTPUT VALIDATION (MANDATORY)

**Output Format**:
```json
{
  "analysis_metadata": {
    "agent_type": "agent_name",
    "estimated_range": {"min": X, "max": Y},
    "actual_count": Z,
    "within_estimate": true,
    "completeness": "100%",
    "status": "COMPLETE"
  },
  "findings_written_to": "findings_file.md",
  "breakdown": { "CRITICAL": N, "HIGH": M, "MEDIUM": L, "LOW": K },
  "validation": {
    "id_sequence_valid": true,
    "no_duplicates": true,
    "all_have_evidence": true,
    "all_have_recommendations": true,
    "within_estimated_range": true
  }
}
```

**VALIDATION CRITERIA**:

✅ `actual_count` within `[min_estimate, max_estimate]` → VALID
✅ If outside range: document `variance_reason` → ACCEPTABLE
❌ Missing required fields (id, file, line, code_snippet, description, recommendation) → INVALID
❌ ID gaps in sequence → SEQUENCE ERROR
❌ Placeholder text ("...", "etc.", "and others") → SUMMARIZATION DETECTED
❌ Statements like "similar issues in X files" → VIOLATION

---

**Version**: 3.0 (AI-Optimized)
**Last Updated**: 2025-10-13

---

END OF COMPLETENESS ENFORCEMENT RULES
