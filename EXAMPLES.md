# FRAMEWORK EXAMPLES - REAL-WORLD WALKTHROUGHS

**Version**: 3.0 (AI-Optimized)
**Purpose**: Concrete execution examples for different codebase sizes
**Last Updated**: 2025-10-13

---

## Example 1: Small Codebase - Standard Strategy

**Project**: invoice-api (Django REST API)
**Size**: 12K LOC, 45 files
**Strategy**: Standard Output (all findings in memory)

### Scenario
```
Languages: Python
Framework: Django 4.2 + PostgreSQL
Architecture: MVC pattern
Expected findings: ~40 total
```

### Execution
```bash
# Phase 1: Discovery
find . -name "*.py" | xargs wc -l  # → 12,340 LOC

# Phase 2: Pattern scan
grep -r "password.*=" --include="*.py"  # → 3 hardcoded passwords
grep -r "eval(" --include="*.py"        # → 1 dangerous eval

# Phase 3: Agent execution (parallel)
Security Agent    → 18 findings (2 CRITICAL, 6 HIGH, 8 MEDIUM, 2 LOW)
Performance Agent → 12 findings (0 CRITICAL, 4 HIGH, 6 MEDIUM, 2 LOW)
Architecture Agent→ 8 findings  (0 CRITICAL, 2 HIGH, 4 MEDIUM, 2 LOW)
TOTAL: 38 findings
```

### Output Strategy
```
Pre-Analysis Estimation: [30, 50] findings (±25%)
Actual Count: 38 ✓ WITHIN RANGE

v3.0 Unified Sampling:
- CRITICAL (2):  ALL detailed
- HIGH (12):     ALL detailed
- MEDIUM (18):   ALL detailed (<20 threshold)
- LOW (6):       ALL detailed (<15 threshold)

Result: 38/38 findings detailed (no Quick Reference Table needed)
Context usage: 45% (safe)
```

---

## Example 2: Large Codebase - Progressive Writing

**Project**: ecommerce-platform (Spring Boot microservices)
**Size**: 138K LOC, 845 files
**Strategy**: Progressive Writing (write to disk during analysis)

### Scenario
```
Languages: Java
Framework: Spring Boot 3.5.5 + Hibernate + Oracle
Architecture: Layered microservice
Expected findings: ~250 total
```

### Execution
```bash
# Phase 1: Discovery → manifest.json
Total LOC: 138,456
Layers: Controllers (42 files), Services (156 files), DAOs (189 files)

# Phase 2: Pattern scan
Hotspots identified: 85 files requiring deep analysis

# Phase 3: Agent execution with Progressive Writing
Security Agent:
  [10%] 25/250 findings written to security_findings.md
  [20%] 50/250 findings written (batch cleared from memory)
  ...
  [100%] 250/250 findings written ✓

Performance Agent: 180 findings written to performance_findings.md
Concurrency Agent: 45 findings written to concurrency_findings.md
JPA Agent: 120 findings written to jpa_findings.md

TOTAL: 595 findings written to disk
```

### Output Strategy
```
Pre-Analysis Estimation: [400, 700] findings (±35%)
Actual Count: 595 ✓ WITHIN RANGE

v3.0 Unified Sampling (applied AFTER all findings written):

security_findings.md:
- CRITICAL (8):  ALL 8 detailed
- HIGH (42):     ALL 42 detailed
- MEDIUM (120):  Top 5 detailed + Quick Ref Table (115 entries)
- LOW (80):      Top 3 detailed + Quick Ref Table (77 entries)

performance_findings.md:
- CRITICAL (0):  -
- HIGH (35):     ALL 35 detailed
- MEDIUM (95):   Top 5 detailed + Quick Ref Table (90 entries)
- LOW (50):      Top 3 detailed + Quick Ref Table (47 entries)

[similar for other domains]

Final report:
- Detailed: 8+42+5+3 + 35+5+3 + ... = 158 findings
- Quick Reference Tables: 437 entries
- Total documented: 595/595 (100%)
- Context usage: Peak 52% (Progressive Writing prevented overflow)
```

---

## Example 3: Very Large Codebase - Strategic Sampling

**Project**: enterprise-erp (Monolithic Java EE)
**Size**: 850K LOC, 4,200 files
**Strategy**: Strategic Sampling (40%) + Progressive Writing

### Scenario
```
Languages: Java, JavaScript, SQL
Framework: Java EE 8 + JSF + EJB
Architecture: Monolithic 3-tier
Coverage target: 40% strategic (100% impossible)
```

### Execution
```bash
# Phase 1: Discovery
Total LOC: 850,124
Total files: 4,218

# Phase 2: Strategic sampling plan
100% coverage: Controllers (180 files)
100% coverage: Security-critical (auth, payment, user mgmt)
80% coverage: Services (350/450 files)
40% coverage: DAOs (300/750 files)
40% coverage: Utilities (200/500 files)

Total analyzed: ~1,800 files (43% coverage)

# Phase 3: Agent execution with Progressive Writing
Security Agent (100% of security-critical paths):
  Analyzed: 520 files
  Findings: 380
  Written to: security_findings.md

Performance Agent (80% of services):
  Analyzed: 350 files
  Findings: 290
  Written to: performance_findings.md

[other agents similar]

TOTAL: 1,245 findings from 1,800 files (43% coverage)
```

### Output Strategy
```
Pre-Analysis Estimation: [800, 1500] findings from 40% sample
Actual Count: 1,245 ✓ WITHIN RANGE

Coverage metadata:
- Entry points: 100% analyzed
- Business logic: 80% analyzed
- Data layer: 40% analyzed
- Utilities: 40% analyzed
- Confidence: HIGH for CRITICAL/HIGH issues
- Confidence: MEDIUM for MEDIUM/LOW issues (sampling may miss some)

v3.0 Unified Sampling (applied to 1,245 findings):
- CRITICAL (45):  ALL detailed
- HIGH (180):     ALL detailed
- MEDIUM (620):   Top 5 detailed + Quick Ref Table (615)
- LOW (400):      Top 3 detailed + Quick Ref Table (397)

Final report:
- Detailed: 233 findings
- Quick Reference: 1,012 entries
- Total documented: 1,245/1,245 (100% of what was found)
- Coverage disclaimer: "43% strategic sampling - prioritized entry points and security paths"
- Context usage: Peak 68% (Progressive Writing + Strategic Sampling)
```

---

## Key Patterns Across Examples

### Pre-Analysis Estimation (v3.0)
- **Small**: Tight range [30, 50] (±25%) - codebase well understood
- **Large**: Wider range [400, 700] (±35%) - more uncertainty
- **Very Large**: Widest range [800, 1500] - sampling adds uncertainty

### Context Management
- **Small**: No special handling (45% usage)
- **Large**: Progressive Writing required (52% peak)
- **Very Large**: Progressive + Strategic Sampling (68% peak)

### Output Sampling
- **ALL examples**: CRITICAL/HIGH = 100% detailed (never sampled)
- **MEDIUM**: Count-based (<20=ALL, 20-50=top 10, >50=top 5)
- **LOW**: Count-based (<15=ALL, 15-40=top 8, >40=top 3)

### Completeness
- **Small**: 38/38 documented (100%)
- **Large**: 595/595 documented (100%)
- **Very Large**: 1,245/1,245 documented (100% of analyzed)
  - Note: Only 43% of codebase analyzed (disclosed in report)

---

## Common Mistakes (Anti-Patterns)

### ❌ Mistake 1: Wrong Strategy Selection
```
BAD:  138K LOC codebase → Standard Strategy → Context overflow at 95% → FAILURE
GOOD: 138K LOC codebase → Progressive Writing → Context 52% → SUCCESS
```

### ❌ Mistake 2: Exact Count Declaration
```
BAD:  "I will find exactly 52 findings" (before analysis)
GOOD: "Estimated range: [36, 77] findings (±35%)"
```

### ❌ Mistake 3: Summarization
```
BAD:  "Found 8 SQL injection vulnerabilities in repository layer"
GOOD: SEC-001, SEC-002, ..., SEC-008 (all 8 listed with file:line)
```

### ❌ Mistake 4: Ignoring Strategic Sampling Threshold
```
BAD:  850K LOC → try 100% analysis → impossible → incomplete
GOOD: 850K LOC → 40% strategic sampling → complete within scope
```

---

**Version**: 3.0 (AI-Optimized)
**Last Updated**: 2025-10-13

---

END OF EXAMPLES
