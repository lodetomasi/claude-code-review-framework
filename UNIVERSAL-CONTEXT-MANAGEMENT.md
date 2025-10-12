# Universal Context Management Instructions - Full Spectrum Analysis

**Version**: 2.3
**Date**: 2025-10-12
**Framework**: claude-code-review-framework
**Purpose**: Smart in-memory compression for context-efficient comprehensive analysis

---

## 📚 READING CONTEXT

**This document is Step 5.5 and 5.6** in the reading order defined in [START-HERE.md](START-HERE.md).

**If you arrived here directly**: Read [START-HERE.md](START-HERE.md) first to understand:
- When to use Universal Context Management (Step 5.5): codebases > 50K LOC
- When to use Progressive Writing Strategy (Step 5.6): codebases > 100K LOC or > 100 expected findings

**This document contains TWO critical strategies**:
1. **Universal Context Management** (Step 5.5): Smart compression for 50-100K LOC
2. **Progressive Writing Strategy** (Step 5.6): Incremental disk writes for >100K LOC

🎯 **[→ GO TO START-HERE.md](START-HERE.md)** if you need to understand when to apply these strategies.

---

## 🎯 FUNDAMENTAL PRINCIPLES
1. **Analyze ALL aspects with EQUAL priority**: Security, Performance, Concurrency, Architecture
2. **Manage context window ACTIVELY**: Monitor usage, compress intelligently, never overflow
3. **Maintain FULL quality**: Complete output regardless of codebase size

---

## ⚠️ CONTEXT WINDOW MANAGEMENT PROTOCOL

### CONTINUOUS MONITORING
**Check your context usage every 10 files analyzed:**

```
AT START:
- Calculate total files and estimated tokens
- If estimated > 150K tokens → Switch to CHUNKED MODE immediately
- If estimated < 50K tokens → FULL MODE
- Between 50K-150K → SMART MODE

DURING ANALYSIS:
- Track tokens used (approximate)
- Monitor remaining capacity
- Adjust strategy dynamically
```

### CONTEXT USAGE THRESHOLDS

**0-60% used**: Normal analysis, full details
**60-70% used**: Start compressing, batch similar findings
**70-80% used**: Critical and High only, minimal evidence
**80-90% used**: Critical only, use pattern codes
**90-95% used**: Stop new analysis, prepare summary
**>95% used**: Emergency output immediately

---

## 📋 MEMORY MANAGEMENT STRATEGY

### PERMANENT MEMORY (Keep until output)
```
CRITICAL findings: Full details for ALL domains
- Security: Vulnerabilities with exploit path
- Performance: Issues causing >5x slowdown
- Concurrency: Data corruption risks
- Architecture: Blockers for maintenance

HIGH findings: Key details only
- Location and pattern type
- Measured impact
- Fix reference
```

### TEMPORARY MEMORY (Process and release)
```
ANALYZE chunk → EXTRACT findings → COMPRESS findings → CLEAR chunk

Never keep in memory:
- Full file contents after analysis
- Duplicate patterns (keep one example)
- Comments and documentation
- Test code (unless has issues)
- Boilerplate code
```

### PROGRESSIVE COMPRESSION
```
First occurrence of pattern: Store full details
Second occurrence: Store location only
Third+ occurrence: Increment counter only

Example:
- N+1 Query #1: Full analysis with evidence
- N+1 Query #2-50: Just locations list
- Output: "N+1 pattern found in 50 locations"
```

---

## 🔄 ADAPTIVE CHUNKING STRATEGY

### SMART SEGMENTATION BASED ON SIZE

**SMALL (<10K LOC)**: Single pass
- Analyze everything in one go
- Full details for all findings
- No compression needed

**MEDIUM (10-50K LOC)**: Layer-based chunks
```
Chunk 1: Entry points (Controllers/APIs)
Chunk 2: Business logic (Services)
Chunk 3: Data layer (Repositories)
Chunk 4: Cross-cutting (Security/Logging)

Between chunks: Compress and clear memory
```

**LARGE (50-100K LOC)**: Pattern-based chunks
```
Chunk 1: Security patterns (scan all files)
Chunk 2: Performance patterns (scan all files)
Chunk 3: Deep dive on hotspots only
Chunk 4: Architecture analysis on core only

Aggressive compression between chunks
```

**VERY LARGE (>100K LOC)**: Sampling mode
```
Sample 20% strategically:
- 100% of entry points
- 50% of business logic
- 20% of utilities
- 100% of security-critical paths

Report sampling confidence
```

---

## 📊 EQUAL-PRIORITY ANALYSIS DOMAINS

### DOMAIN 1: SECURITY (25% attention)
**Impact**: Data breaches, compliance failures
```
ALWAYS CHECK:
- Input validation gaps
- Authentication/authorization flaws
- Injection vulnerabilities
- Cryptographic weaknesses
- Sensitive data exposure
```

### DOMAIN 2: PERFORMANCE (25% attention)
**Impact**: User experience, scalability limits
```
ALWAYS CHECK:
- N+1 query problems
- Missing database indexes
- Algorithm complexity (O(n²)+)
- Memory inefficiencies
- Synchronous operations blocking threads
- Missing caching opportunities
```

### DOMAIN 3: CONCURRENCY (25% attention)
**Impact**: Data corruption, system crashes
```
ALWAYS CHECK:
- Race conditions
- Deadlock patterns
- Thread pool exhaustion
- Non-thread-safe operations
- Missing synchronization
```

### DOMAIN 4: ARCHITECTURE (25% attention)
**Impact**: Maintenance burden, team velocity
```
ALWAYS CHECK:
- Circular dependencies
- God classes/modules
- SOLID violations
- Layer mixing
- Tight coupling
```

---

## 🎯 CONTEXT-AWARE FINDING STORAGE

### COMPRESSION FORMATS BY CONTEXT USAGE

**FULL FORMAT (0-60% context)**:
```json
{
  "id": "PERF-CRIT-001",
  "type": "PERFORMANCE",
  "severity": "CRITICAL",
  "category": "N_PLUS_ONE",
  "file": "OrderService.java",
  "line": 234,
  "evidence": "orders.forEach(o -> o.getItems().size())",
  "description": "N+1 query loading items for each order",
  "impact": "500 orders generate 501 queries",
  "fix": "Use JOIN FETCH or batch loading",
  "code_example": "@Query('SELECT o FROM Order o JOIN FETCH o.items')"
}
```

**COMPRESSED FORMAT (60-80% context)**:
```json
{
  "id": "PERF-C-001",
  "pattern": "N+1",
  "loc": "OrderService:234",
  "impact": "501 queries for 500 orders",
  "fix_ref": "JOIN_FETCH_PATTERN"
}
```

**MINIMAL FORMAT (80-95% context)**:
```
PERF-C:OrderService:234:N+1:501q
```

---

## 🔄 INTELLIGENT BATCHING RULES

### ALWAYS batch similar findings:

**Instead of**:
```
10 separate N+1 findings = 10 × 200 tokens = 2000 tokens
```

**Use**:
```
1 batched finding = 300 tokens
"N+1 Query Pattern:
 - Instances: 10
 - Locations: [Service:23, Service:45, DAO:67...]
 - Total excess queries: 5000
 - Fix pattern: Apply batch loading globally"
```

---

## 📈 PERFORMANCE-SPECIFIC PATTERNS

### MUST CHECK in ANY language:

**Database/Data Access**:
```
N+1 QUERIES: Loop with individual queries
MISSING INDEX: WHERE/JOIN without index
FULL SCAN: SELECT * without limit
NO PAGINATION: Loading all records
INEFFICIENT JOIN: Cartesian products
```

**Algorithms/Processing**:
```
NESTED LOOPS: O(n²)+ with large datasets
INEFFICIENT SORT: Bubble sort on large data
REPEATED COMPUTATION: Same calculation in loop
STRING CONCATENATION: In loops (StringBuilder needed)
```

**Resource Usage**:
```
MEMORY WASTE: Large collections in memory
NO STREAMING: Loading files entirely
CONNECTION LEAKS: Unclosed resources
THREAD WASTE: Creating threads per request
```

---

## 📊 ANALYSIS DEPTH CONTROL

### Based on context remaining:

**PLENTY OF CONTEXT (>40% remaining)**:
- Deep analysis of all issues
- Full code examples
- Detailed fix instructions
- Alternative solutions

**LIMITED CONTEXT (20-40% remaining)**:
- Focus on CRITICAL/HIGH only
- Essential fix information
- One-line code hints
- Reference to patterns

**MINIMAL CONTEXT (<20% remaining)**:
- CRITICAL only
- Pattern name and location
- Fix reference code
- Count of similar issues

---

## 🎬 FINAL OUTPUT RECONSTRUCTION

### ALWAYS expand compressed findings in final output:

**During analysis**: Store compressed
**Final output**: Expand to full detail

```
STORED (compressed): "PERF-C:Order:234:N+1:501q"

OUTPUT (expanded):
"### CRITICAL: N+1 Query Problem
**Location**: OrderService.java, line 234
**Problem**: Loading order items in loop causes 501 database queries
**Impact**: Response time degraded by 15 seconds
**Solution**: Implement JOIN FETCH pattern
**Example**: [full code example]
**Effort**: 2 hours to implement and test"
```

---

## 🚦 SMART SAMPLING STRATEGIES

### When context is limited:

**SECURITY**: Sample 100% (never skip)
**PERFORMANCE**: Sample hot paths + database operations
**CONCURRENCY**: Sample shared state + thread operations
**ARCHITECTURE**: Sample core modules + interfaces

### Sampling confidence reporting:
```
"Analysis Coverage:
- Security: 100% analyzed
- Performance: 85% analyzed (hot paths covered)
- Concurrency: 90% analyzed (all thread operations)
- Architecture: 60% analyzed (core modules only)
- Confidence: HIGH for critical issues"
```

---

## ✅ QUALITY PRESERVATION CHECKLIST

**Before outputting, verify**:
- [ ] All CRITICAL issues found and detailed
- [ ] Performance issues have metrics
- [ ] Concurrency issues have probability assessment
- [ ] Architecture issues have refactoring effort
- [ ] Context usage stayed below 95%
- [ ] All compressed findings expanded
- [ ] Coverage percentage reported

---

## 🔴 ABSOLUTE RULES

**NEVER**:
1. Skip security analysis to save context
2. Ignore performance for more security analysis
3. Exceed 95% context usage
4. Output compressed format to user
5. Lose findings between chunks
6. Analyze test files unless critical

**ALWAYS**:
1. Balance all domains equally
2. Monitor context usage actively
3. Compress during analysis
4. Expand for final output
5. Report coverage percentage
6. Prioritize by actual impact

---

## 📊 EXPECTED OUTCOMES

| Codebase Size | Context Strategy | Coverage | Output Quality |
|---------------|-----------------|----------|----------------|
| <10K LOC | Full analysis | 100% | Complete |
| 10-50K LOC | Smart chunks | 95% | Complete |
| 50-100K LOC | Compressed | 85% | Complete for Critical/High |
| >100K LOC | Sampling | 60% | Complete for sampled portions |

---

## 💡 CONTEXT OPTIMIZATION TECHNIQUES

### 1. **Reference-based patterns**:
```
Instead of: "SQL injection via string concatenation in query construction"
Use: "SQLI-CONCAT"
```

### 2. **Incremental counters**:
```
Instead of: Listing all 50 occurrences
Use: "Pattern found: 50 instances across 12 files"
```

### 3. **Cross-reference indices**:
```
Instead of: Repeating relationship descriptions
Use: "See ARCH-001" (reference earlier finding)
```

### 4. **Pattern libraries**:
```
Define once: "N1-PATTERN: N+1 query problem"
Reference: "Found N1-PATTERN at line 234"
```

---

## 🚀 PROGRESSIVE WRITING STRATEGY (v2.3)

### Problem

Large codebases generate 800+ findings → Output overflow (>32K token limit)

### Solution

**Write findings incrementally to disk during analysis, not at the end**

### Implementation

#### Phase 1: Category-Based Progressive Analysis

Launch 4 agents in parallel, each writes to its own file:

```bash
Agent 1 → security_findings.md
Agent 2 → performance_findings.md
Agent 3 → concurrency_findings.md
Agent 4 → architecture_findings.md
```

#### Phase 2: Incremental Writing Pattern

Each agent follows this pattern:

```bash
# Initialize file with header
cat > security_findings.md <<'EOF'
## Security Issues
### CRITICAL Issues
EOF

# Analysis loop
findings_batch=()
count=0

for file in all_files:
    issues = analyze_file(file)
    findings_batch.extend(issues)
    count += len(issues)

    # Every 50 findings, FLUSH TO DISK
    if count % 50 == 0:
        for finding in findings_batch:
            write_to_file(finding)

        # CLEAR from context (critical!)
        findings_batch = []

        print(f"[Progress] {count} findings written")

# Write remaining
for finding in findings_batch:
    write_to_file(finding)
```

#### Phase 3: Sampling Application

After writing ALL findings, apply sampling:

```python
def apply_sampling(findings_file):
    """
    Keep:
    - ALL CRITICAL
    - ALL HIGH
    - Top 30% MEDIUM (by impact)
    - Top 20% LOW (by frequency)
    """

    findings = read_all_findings(findings_file)

    critical = filter(severity == 'CRITICAL')
    high = filter(severity == 'HIGH')
    medium = filter(severity == 'MEDIUM')
    low = filter(severity == 'LOW')

    # Keep all CRIT + HIGH
    kept = critical + high

    # Sample MEDIUM: top 30%
    medium_sorted = sort_by_impact(medium)
    kept += medium_sorted[:int(len(medium) * 0.30)]

    # Sample LOW: top 20%
    low_grouped = group_by_pattern(low)
    for pattern, instances in low_grouped:
        kept += instances[:int(len(instances) * 0.20)]

    # Overwrite file with sampled findings
    write_findings_file(findings_file, kept)

    return len(kept)
```

#### Phase 4: Merge

```bash
cat security_findings.md \
    performance_findings.md \
    concurrency_findings.md \
    architecture_findings.md \
    > CODE_REVIEW_REPORT_v2.2.md
```

### Benefits

| Metric | Without Progressive Write | With Progressive Write |
|--------|---------------------------|------------------------|
| Peak Context | 150KB (crash) | 50KB (safe) |
| Output Size | >32K tokens (fails) | 25K tokens (success) |
| Issues Found | 0 (crashed) | 800+ (found all) |
| Issues Documented | 0 | 220 (sampled) |
| Context Savings | N/A | **67%** |

### Example: Security Analysis

```
Files analyzed: 1,350 Java files
Issues found: 250 total

Batch 1 (files 1-100):     28 issues → Write to disk → Clear
Batch 2 (files 101-200):   35 issues → Write to disk → Clear
Batch 3 (files 201-300):   42 issues → Write to disk → Clear
...
Batch 13 (files 1201-1350): 18 issues → Write to disk → Clear

Total written: 250 issues to security_findings.md

Apply sampling:
- CRITICAL: 8 → Keep ALL (8)
- HIGH: 42 → Keep ALL (42)
- MEDIUM: 120 → Keep 30% (36)
- LOW: 80 → Keep 20% (16)

Final: 102 issues in security_findings.md
```

### Agent Instructions

When implementing this, agents must:

1. **Initialize output file** at start
2. **Accumulate findings** in batches of 50
3. **Write batch to file** every 50 findings
4. **Clear batch from context** after writing
5. **Continue analysis** with freed context
6. **Apply sampling** at the end
7. **Report statistics** (found vs documented)

### Output Format

Each finding written to file:

```markdown
### SEC-042: Weak MD5 Hashing
**File**: `UserService.java:123`
**Problem**: Using MD5 for password hashing
**Fix**: Replace with BCrypt

---
```

---

## 🎯 SUCCESS CRITERIA

Your analysis succeeds when:
1. **No context overflow** (stayed under 95%)
2. **All domains analyzed** (security, performance, concurrency, architecture)
3. **Output is complete** (all findings properly detailed)
4. **Coverage is clear** (user knows what was/wasn't analyzed)
5. **Fixes are actionable** (developer can implement immediately)
6. **Progressive writing used** (for >100K LOC codebases)
7. **Sampling applied** (CRIT=ALL, HIGH=ALL, MED=30%, LOW=20%)

---

## REMEMBER

**During analysis**: Compress, batch, reference, forget
**In final output**: Expand, detail, exemplify, complete
**Context is limited**: But output quality is not negotiable
**All aspects matter**: Security = Performance = Concurrency = Architecture

---

## Integration with Framework

This document should be read by:
- **Orchestrator AI**: Before launching analysis
- **Specialized Agents**: Before starting domain analysis
- **Report Generators**: To understand compression patterns

See also:
- `START-HERE.md`: Reading guide
- `AGENT-PROMPTS.md`: Agent templates
- `CLAUDE-ANALYSIS-FRAMEWORK.md`: Overall workflow
- `COMPLETENESS-ENFORCEMENT.md`: Quality rules
