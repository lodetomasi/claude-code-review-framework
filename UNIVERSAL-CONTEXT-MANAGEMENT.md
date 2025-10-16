# UNIVERSAL CONTEXT MANAGEMENT - AI EXECUTION RULES

**Version**: 3.0 (AI-Optimized)
**Mandatory Reading**: Step 3 in execution sequence
**Last Updated**: 2025-10-13

---

## ⛔ ABSOLUTE PROHIBITIONS - CONTEXT MANAGEMENT

**VIOLATION = CONTEXT OVERFLOW → ANALYSIS FAILS**

1. ❌ **FORBIDDEN** to exceed 95% context usage without activating Progressive Writing
2. ❌ **FORBIDDEN** to keep all findings in memory instead of using write-clear-continue pattern
3. ❌ **FORBIDDEN** to use Standard Strategy on codebases >100K LOC
4. ❌ **FORBIDDEN** to compress or summarize CRITICAL/HIGH severity findings
5. ❌ **FORBIDDEN** to skip context monitoring (must check every 10 files)
6. ❌ **FORBIDDEN** to continue analysis after context overflow warning
7. ❌ **FORBIDDEN** to batch write findings without clearing memory afterward
8. ❌ **FORBIDDEN** to ignore dynamic write interval adjustments
9. ❌ **FORBIDDEN** to analyze entire codebase without LOC count first
10. ❌ **FORBIDDEN** to choose strategy without considering estimated finding count

---

## 🚨 FATAL ERRORS - CONTEXT VIOLATIONS

### FATAL-201: Context Overflow
- **Condition**: Context usage exceeds 95% without activating Progressive Writing
- **Consequence**: Analysis FAILS - findings lost to truncation
- **Recovery**: Switch to Progressive Writing immediately, initialize output files

### FATAL-202: Findings Lost to Compression
- **Condition**: CRITICAL or HIGH findings compressed during analysis
- **Consequence**: Analysis INCOMPLETE - critical issues missing
- **Recovery**: Re-analyze with proper memory management, never compress high-severity

### FATAL-203: Wrong Strategy for Codebase Size
- **Condition**: Standard Strategy used on codebase >100K LOC or >100 expected findings
- **Consequence**: GUARANTEED FAILURE - context overflow inevitable
- **Recovery**: Calculate LOC correctly, switch to Progressive Writing, restart analysis

### FATAL-204: Write Without Clear Pattern
- **Condition**: Findings written to disk but not cleared from memory
- **Consequence**: Memory accumulation → context overflow
- **Recovery**: Implement write-then-clear pattern: write batch, clear list, continue

---

## RULE HIERARCHY (V3.0)

**Priority Order**: COMPLETENESS > CONTEXT MANAGEMENT > OUTPUT STRATEGY

**Compression Rules**:
- During analysis: compress in memory (temporary)
- Write to disk: expand ALL findings (permanent)
- Final output: apply count-based sampling (presentation)

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

### PROGRESSIVE COMPRESSION (During Analysis Only)
```
First occurrence of pattern: Store full details
Second occurrence: Store location only
Third+ occurrence: Increment counter only

Example (IN-MEMORY during analysis):
- N+1 Query #1: Full analysis with evidence
- N+1 Query #2-50: Just locations list (compressed)

Example (DISK WRITE via Progressive Writing):
- Write ALL 50 occurrences to disk with full details
- Clear from memory after writing

Example (FINAL OUTPUT per v3.0 Unified Strategy):
- If CRITICAL/HIGH: Keep ALL 50 in report
- If MEDIUM and >20 total: Top 10 detailed + Quick Reference Table for remaining 40
- If LOW and >15 total: Top 3 detailed + Quick Reference Table for remaining 47

**CRITICAL RULE**: NEVER compress CRITICAL or HIGH findings - always full details
```

---

## 🔄 ADAPTIVE CHUNKING STRATEGY

| Codebase Size | Strategy | Coverage | Technique |
|---------------|----------|----------|-----------|
| **<10K LOC** | Single pass | 100% | Analyze all, full details, no compression |
| **10-50K LOC** | Layer-based chunks | 100% | Chunk by layer (Controllers→Services→DAOs→Cross-cutting), compress between |
| **50-100K LOC** | Pattern-based chunks | 100% | Chunk by pattern (Security→Performance→Hotspots→Architecture), aggressive compression |
| **>100K LOC or >100 findings** | Progressive Writing | 100% | Write to disk every N findings (dynamic), clear from memory, full coverage |
| **>500K LOC** | Strategic Sampling | 40%+ | 100% entry points, 80% business logic, 40% data/utilities, report confidence |

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

| Context Usage | Format | Example |
|---------------|--------|---------|
| **0-60%** | Full | All fields (id, type, severity, category, file, line, evidence, description, impact, fix, code_example) |
| **60-80%** | Compressed | Essential fields (id, pattern, loc, impact, fix_ref) |
| **80-95%** | Minimal | String: `PERF-C:OrderService:234:N+1:501q` |

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

## 📊 EXPECTED OUTCOMES (v3.0)

| Codebase Size | Context Strategy | Analysis Coverage | Output Strategy |
|---------------|-----------------|-------------------|-----------------|
| <10K LOC | Full analysis | 100% | All findings detailed |
| 10-50K LOC | Smart chunks | 100% | All findings detailed |
| 50-100K LOC | Progressive Writing | 100% | v3.0 Unified (count-based) |
| 100-500K LOC | Progressive Writing | 100% | v3.0 Unified (count-based) |
| >500K LOC | Strategic Sampling | 40%+ | v3.0 Unified (count-based) |

**Key Change from v2.3**: 100K-500K LOC now get 100% analysis coverage (not 60%)

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

## 🚀 PROGRESSIVE WRITING STRATEGY (v3.0)

**Problem**: Large codebases generate 800+ findings → Output overflow (>32K token limit)

**Solution**: Write findings to disk during analysis (not at end) using write-clear-continue pattern

**Implementation**:
1. Launch agents in parallel → each writes to `{domain}_findings.md`
2. Each agent: analyze files → accumulate batch → write to disk → clear from memory
3. Dynamic write intervals (see SAMPLING-RULES.md): context <70%=50 findings, 70-85%=25, 85-95%=10, >95%=1
4. After ALL findings written → apply v3.0 Unified Sampling (SAMPLING-RULES.md)
5. Merge domain files → `CODE_REVIEW_REPORT_v3.0.md`

**Benefits**: Peak context 50KB (vs 150KB crash), 800+ findings documented, 67% context savings

**See EXAMPLES.md Example 2 for complete Progressive Writing walkthrough**

---

## 🎯 SUCCESS CRITERIA (v3.0)

Your analysis succeeds when:
1. **No context overflow** (stayed under 95%)
2. **All domains analyzed** (security, performance, concurrency, architecture)
3. **Output is complete** (all findings found and written to disk)
4. **Coverage is clear** (user knows what was/wasn't analyzed)
5. **Fixes are actionable** (developer can implement immediately)
6. **Progressive writing used** (for >100K LOC codebases)
7. **v3.0 Unified Sampling applied** (count-based rules):
   - CRITICAL/HIGH: ALL (100%)
   - MEDIUM: ALL if <20, else top samples + Quick Ref
   - LOW: ALL if <15, else top samples + Quick Ref
8. **100% documented** (via combination of detailed + Quick Reference Tables)

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
