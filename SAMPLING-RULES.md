# SAMPLING RULES - CANONICAL SPECIFICATION

**Version**: 3.0 (AI-Optimized)
**Status**: Single Source of Truth for Sampling Logic
**Last Updated**: 2025-10-13

---

## ⚠️ CRITICAL - CANONICAL REFERENCE

**This document is the ONLY authoritative source for sampling rules.**

All other framework documents MUST reference this file, not duplicate these rules.

---

## 🎯 COUNT-BASED SAMPLING RULES (v3.0)

### CRITICAL Findings

```text
Rule: Keep ALL in detailed format
Count: 100% (never sampled)
Reason: Immediate security/data corruption risks
Output: Detailed format (5 lines each)
```

### HIGH Findings

```text
Rule: Keep ALL in detailed format
Count: 100% (never sampled)
Reason: Significant impact on system
Output: Detailed format (5 lines each)
```

### MEDIUM Findings (Count-Based)

```text
IF count < 20:
  → Keep ALL detailed
  → No Quick Reference Table needed

ELIF count between 20-50:
  → Keep top 10 detailed (sorted by impact)
  → Remaining in Quick Reference Table

ELIF count > 50:
  → Keep top 5 detailed (sorted by impact)
  → Remaining in Quick Reference Table
```

**Sorting Criteria**: Impact score (exploitability × business consequence)

### LOW Findings (Count-Based)

```text
IF count < 15:
  → Keep ALL detailed
  → No Quick Reference Table needed

ELIF count between 15-40:
  → Keep top 8 detailed (sorted by frequency)
  → Remaining in Quick Reference Table

ELIF count > 40:
  → Keep top 3 detailed (sorted by frequency)
  → Remaining in Quick Reference Table
```

**Sorting Criteria**: Frequency (most common patterns first)

---

## 📊 QUICK REFERENCE TABLE FORMAT

When sampling is applied, remaining findings MUST be documented in Quick Reference Table:

```markdown
| ID | Severity | Category | File:Line | Brief Description |
|----|----------|----------|-----------|-------------------|
| SEC-051 | MEDIUM | INPUT_VALIDATION | AuthController.java:45 | Missing @Valid annotation |
| SEC-052 | MEDIUM | WEAK_CRYPTO | UserService.java:123 | MD5 used instead of BCrypt |
```

**Table Requirements**:

- ALL findings included (100% documentation)
- One line per finding
- Hyperlink to file:line if possible
- Brief description (max 60 chars)

---

## 🔄 DYNAMIC WRITE INTERVALS (v3.0)

### Progressive Writing Strategy

**Purpose**: Prevent context overflow during analysis of large codebases

**Algorithm**:
```python
def calculate_write_interval(context_usage_percentage):
    """
    Dynamic write interval based on current context usage.

    Returns: Number of findings to accumulate before writing to disk
    """
    if context_usage_percentage < 70:
        return 50  # Low pressure: batch 50 findings

    elif 70 <= context_usage_percentage < 85:
        return 25  # Moderate pressure: batch 25 findings

    elif 85 <= context_usage_percentage < 95:
        return 10  # High pressure: batch 10 findings

    else:  # context_usage >= 95
        return 1   # Critical: write immediately (every finding)
```

**Usage Pattern**:
```python
findings_batch = []
total_written = 0

for file in all_files:
    issues = analyze_file(file)
    findings_batch.extend(issues)

    # Calculate dynamic interval
    context_pct = get_context_usage_percentage()
    write_interval = calculate_write_interval(context_pct)

    # Flush to disk when threshold reached
    if len(findings_batch) >= write_interval:
        write_to_disk(findings_batch)
        total_written += len(findings_batch)
        findings_batch = []  # CLEAR MEMORY - CRITICAL!

        print(f"[Progress] {total_written} findings written (interval: {write_interval})")

# Write remaining batch
if findings_batch:
    write_to_disk(findings_batch)
    total_written += len(findings_batch)
```

---

## 📋 PRE-ANALYSIS ESTIMATION (v3.0)

### Confidence Interval Approach

**Rule**: Declare RANGE [min, max] before analysis, not exact count

**Rationale**: Impossible to know exact count before analyzing code

**Format**:
```json
{
  "pre_analysis_estimation": {
    "estimated_range": {"min": 36, "max": 77},
    "confidence_level": "±35%",
    "estimation_method": "grep scan + extrapolation",
    "files_to_analyze": 845,
    "categories": {
      "SECURITY": {"min": 10, "max": 25},
      "PERFORMANCE": {"min": 15, "max": 30},
      "CONCURRENCY": {"min": 5, "max": 12},
      "ARCHITECTURE": {"min": 6, "max": 10}
    }
  }
}
```

### Validation Rules

**Post-Analysis Validation**:
```python
def validate_estimation(estimated_min, estimated_max, actual_count):
    """
    Validate that actual count falls within estimated range.

    Returns: (is_valid, variance_percentage)
    """
    if estimated_min <= actual_count <= estimated_max:
        midpoint = (estimated_min + estimated_max) / 2
        variance = ((actual_count - midpoint) / midpoint) * 100
        return (True, variance)
    else:
        # Outside range: document reason
        if actual_count < estimated_min:
            reason = "fewer_findings_than_expected"
        else:
            reason = "more_findings_than_expected"
        return (False, reason)
```

**Acceptable Outcomes**:
- ✅ `actual_count` within `[min, max]` → VALID
- ✅ `actual_count` outside range + documented reason → ACCEPTABLE
- ❌ `actual_count` outside range + no reason → INVALID

---

## 🚨 VALIDATION FORMULAS

### Output Validation Checklist

```python
def validate_agent_output(agent_output):
    """
    Validate agent output against sampling rules.

    Returns: (is_valid, errors)
    """
    errors = []

    # Check 1: CRITICAL findings all detailed
    if agent_output['breakdown']['CRITICAL']['detailed'] != agent_output['breakdown']['CRITICAL']['found']:
        errors.append("FATAL: Not all CRITICAL findings detailed")

    # Check 2: HIGH findings all detailed
    if agent_output['breakdown']['HIGH']['detailed'] != agent_output['breakdown']['HIGH']['found']:
        errors.append("FATAL: Not all HIGH findings detailed")

    # Check 3: MEDIUM sampling correct
    medium_count = agent_output['breakdown']['MEDIUM']['found']
    medium_detailed = agent_output['breakdown']['MEDIUM']['detailed']

    if medium_count < 20:
        if medium_detailed != medium_count:
            errors.append(f"MEDIUM count {medium_count} < 20: ALL should be detailed, got {medium_detailed}")
    elif 20 <= medium_count <= 50:
        if medium_detailed != 10:
            errors.append(f"MEDIUM count {medium_count} in [20-50]: Expected 10 detailed, got {medium_detailed}")
    else:  # medium_count > 50
        if medium_detailed != 5:
            errors.append(f"MEDIUM count {medium_count} > 50: Expected 5 detailed, got {medium_detailed}")

    # Check 4: LOW sampling correct
    low_count = agent_output['breakdown']['LOW']['found']
    low_detailed = agent_output['breakdown']['LOW']['detailed']

    if low_count < 15:
        if low_detailed != low_count:
            errors.append(f"LOW count {low_count} < 15: ALL should be detailed, got {low_detailed}")
    elif 15 <= low_count <= 40:
        if low_detailed != 8:
            errors.append(f"LOW count {low_count} in [15-40]: Expected 8 detailed, got {low_detailed}")
    else:  # low_count > 40
        if low_detailed != 3:
            errors.append(f"LOW count {low_count} > 40: Expected 3 detailed, got {low_detailed}")

    # Check 5: 100% documentation
    total_found = sum(agent_output['breakdown'][sev]['found'] for sev in ['CRITICAL', 'HIGH', 'MEDIUM', 'LOW'])
    total_documented = sum(agent_output['breakdown'][sev]['detailed'] + agent_output['breakdown'][sev]['in_table'] for sev in ['CRITICAL', 'HIGH', 'MEDIUM', 'LOW'])

    if total_found != total_documented:
        errors.append(f"FATAL: Found {total_found} but documented {total_documented} (completeness violation)")

    return (len(errors) == 0, errors)
```

---

## 📖 USAGE GUIDELINES

### For Agent Developers

**When writing agents**:
1. Import sampling logic from this file
2. Apply count-based rules AFTER finding all issues
3. Never sample during analysis phase (find 100% first)
4. Validate output using validation formulas above

### For Framework Users

**When configuring analysis**:
1. Agents automatically follow these rules
2. If unsure about sampling: use Progressive Writing (never fails)
3. Adjust thresholds only for specific project requirements

### For Documentation Authors

**When writing framework docs**:
1. Reference `SAMPLING-RULES.md` instead of duplicating
2. Use this exact phrasing: `"See SAMPLING-RULES.md for complete specification"`
3. Never copy-paste sampling logic into other docs

---

## 🔗 INTEGRATION WITH FRAMEWORK

### Documents That Reference This File

**Core Documents**:
- `COMPLETENESS-ENFORCEMENT.md` → References estimation validation
- `UNIVERSAL-CONTEXT-MANAGEMENT.md` → References dynamic write intervals
- `AGENT-PROMPTS.md` → References count-based sampling
- `FRAMEWORK-RULES-HIERARCHY.md` → References as Priority 3 specification

**Agent Templates**:
- Security Agent → Applies count-based sampling
- Performance Agent → Applies count-based sampling
- Concurrency Agent → Applies count-based sampling
- Architecture Agent → Applies count-based sampling

---

## 🎯 SUCCESS METRICS

After applying these rules, validate:
- ✅ All CRITICAL findings detailed (100%)
- ✅ All HIGH findings detailed (100%)
- ✅ MEDIUM sampling follows count-based rules
- ✅ LOW sampling follows count-based rules
- ✅ Quick Reference Table present when sampling applied
- ✅ Total documented = Total found (100% completeness)
- ✅ Context usage remained under 95%

---

## ⚠️ VIOLATION CONSEQUENCES

**If sampling rules violated**:
- Agent output REJECTED
- Analysis marked INVALID
- Must re-run analysis with correct sampling

**Common Violations**:
- ❌ Sampling CRITICAL findings (FATAL)
- ❌ Sampling HIGH findings (FATAL)
- ❌ Wrong thresholds (e.g., sampling MEDIUM at count=15 instead of 20)
- ❌ Missing Quick Reference Table when required
- ❌ Documented count ≠ Found count (completeness violation)

---

**Version**: 3.0 (AI-Optimized)
**Last Updated**: 2025-10-13
**Status**: Canonical Reference

---

END OF SAMPLING RULES SPECIFICATION
