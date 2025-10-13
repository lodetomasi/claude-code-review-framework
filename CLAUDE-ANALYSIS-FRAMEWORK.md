# CLAUDE CODE ANALYSIS FRAMEWORK - AI EXECUTION WORKFLOW

**Version**: 3.0 (AI-Optimized)
**Mandatory Reading**: Step 4 in execution sequence
**Last Updated**: 2025-10-13

---

## ⛔ ABSOLUTE PROHIBITIONS - WORKFLOW EXECUTION

**VIOLATION = WORKFLOW INVALID - ANALYSIS REJECTED**

1. ❌ **FORBIDDEN** to skip Phase 1 Discovery (manifest generation mandatory)
2. ❌ **FORBIDDEN** to skip Phase 2 Pattern Scanning (hotspot identification mandatory)
3. ❌ **FORBIDDEN** to execute agents without Phase 0 briefing (completeness rules)
4. ❌ **FORBIDDEN** to skip agent output validation (Phase 5 mandatory)
5. ❌ **FORBIDDEN** to proceed without reading COMPLETENESS-ENFORCEMENT.md first
6. ❌ **FORBIDDEN** to analyze code without semantic layer segmentation
7. ❌ **FORBIDDEN** to skip deduplication in assembly phase
8. ❌ **FORBIDDEN** to omit manifest.json in agent context
9. ❌ **FORBIDDEN** to run agents sequentially when parallel execution possible
10. ❌ **FORBIDDEN** to skip final validation before report generation

---

## 🚨 FATAL ERRORS - WORKFLOW FAILURES

### FATAL-301: Discovery Phase Skipped
- **Condition**: Agents executed without manifest.json
- **Consequence**: No architectural context - findings INCOMPLETE
- **Recovery**: Run Phase 1 Discovery, generate manifest, re-execute agents

### FATAL-302: Pattern Scanning Skipped
- **Condition**: Deep analysis without hotspot pre-scan
- **Consequence**: Inefficient analysis - may miss critical issues
- **Recovery**: Run Phase 2 Pattern Scanning, prioritize hotspots

### FATAL-303: Validation Phase Skipped
- **Condition**: Agent outputs not validated against completeness rules
- **Consequence**: Invalid findings - output cannot be trusted
- **Recovery**: Run validation checks, reject invalid agent outputs

---

## FRAMEWORK CAPABILITIES

**Semantic Segmentation** | **Specialized Agents** | **Pattern Scanning** | **Tiered Analysis** | **Language Plugins** | **Hash Deduplication** | **Dependency Graph** | **Size-Based Routing**

---

## ARCHITECTURE

```
┌─────────────────────────────────────────────────────────────┐
│                    ORCHESTRATOR                              │
│  - Discovery Phase                                           │
│  - Manifest Generation                                       │
│  - Agent Coordination                                        │
│  - Result Assembly & Deduplication                           │
└─────────────────────────────────────────────────────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   SECURITY   │  │ PERFORMANCE  │  │ CONCURRENCY  │
│    AGENT     │  │    AGENT     │  │    AGENT     │
└──────────────┘  └──────────────┘  └──────────────┘
        │                  │                  │
        ▼                  ▼                  ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│  JPA/ORM     │  │  RESILIENCE  │  │ ARCHITECTURE │
│   AGENT      │  │    AGENT     │  │    AGENT     │
└──────────────┘  └──────────────┘  └──────────────┘
        │                  │                  │
        └──────────────────┼──────────────────┘
                           ▼
                  ┌──────────────┐
                  │   ASSEMBLY   │
                  │   - Merge    │
                  │   - Dedupe   │
                  │   - Prioritize│
                  └──────────────┘
                           │
                           ▼
                  ┌──────────────┐
                  │  FINAL       │
                  │  REPORT      │
                  └──────────────┘
```

---

## WORKFLOW

### Phase 0: Agent Instruction Briefing

**CRITICAL**: All agents MUST follow the **COMPLETENESS ENFORCEMENT RULES v3.0** defined in `COMPLETENESS-ENFORCEMENT.md`.

**Three-Phase Execution (v3.0)**:
1. **Pre-Analysis ESTIMATION**: Agents estimate finding count range [min, max] with confidence level
2. **Progressive Extraction**: Report progress every 10% with specific finding IDs
3. **Output Validation**: Ensure `actual_count` within `[min_estimate, max_estimate]` OR document variance

**Key Requirement**: Document EVERY finding individually - NO summarization or grouping statements like "8 SQL injection vulnerabilities found".

**Rule Hierarchy**: When conflicts arise, follow priority order in `FRAMEWORK-RULES-HIERARCHY.md`:
1. COMPLETENESS (find all) > 2. CONTEXT MANAGEMENT (compression technique) > 3. OUTPUT STRATEGY (presentation format)

**See**: `COMPLETENESS-ENFORCEMENT.md` for full specification and `AGENT-PROMPTS.md` for integrated agent templates.

---

### Phase 1: Discovery (5-10 minutes)

**Goal**: Map the entire codebase structure without analyzing content

**Steps**:

1. **Directory Tree Scan**
```bash
find . -type f -name "*.java" -o -name "*.py" -o -name "*.js" | wc -l
tree -L 3 -I 'node_modules|target|build|dist' > structure.txt
```

2. **Technology Detection**
```bash
# Detect languages
find . -name "*.java" | head -1  # Java
find . -name "pom.xml" | head -1  # Maven
find . -name "requirements.txt" | head -1  # Python pip
find . -name "package.json" | head -1  # Node.js

# Detect frameworks
grep -r "springframework" --include="*.xml" --include="*.java" -l | head -1
grep -r "from flask import" --include="*.py" -l | head -1
grep -r "express()" --include="*.js" -l | head -1
```

3. **Layer Identification**
```bash
# Identify architectural layers
find . -type d -name "controller*" -o -name "service*" -o -name "repository*" -o -name "dao*"
```

4. **Generate Manifest**
```json
{
  "project_name": "detected-from-pom-or-package-json",
  "languages": ["java", "xml", "yaml"],
  "frameworks": ["spring-boot", "hibernate", "feign"],
  "databases": ["oracle", "postgresql"],
  "total_files": 845,
  "total_loc": 125340,
  "architecture": {
    "pattern": "layered-microservice",
    "layers": {
      "controllers": {
        "path": "src/main/java/**/controller",
        "files": 42,
        "loc": 8430
      },
      "services": {
        "path": "src/main/java/**/service",
        "files": 156,
        "loc": 45200
      },
      "dao": {
        "path": "src/main/java/**/dao",
        "files": 189,
        "loc": 32100
      },
      "integration": {
        "path": "src/main/java/**/integration",
        "files": 24,
        "loc": 6800
      },
      "util": {
        "path": "src/main/java/**/util",
        "files": 38,
        "loc": 5200
      }
    }
  },
  "hotspots": {
    "large_files": [
      {"file": "path/to/large.java", "loc": 2500},
      {"file": "path/to/huge.java", "loc": 3200}
    ],
    "high_complexity": [
      {"file": "path/to/complex.java", "cyclomatic": 45}
    ]
  },
  "dependencies": {
    "external": ["spring-boot:3.5.5", "hibernate:6.2", "feign:4.0"],
    "internal_modules": ["module-a", "module-b"]
  }
}
```

**Output**: `manifest.json` - Complete structural map of the codebase

---

### Phase 2: Pattern Scanning (Quick Hotspot Identification)

**Goal**: Use grep/ripgrep to find high-priority patterns WITHOUT reading entire files

**Examples**:

```bash
# Security patterns
grep -r "password.*=" --include="*.java" --include="*.properties" -n
grep -r "Runtime.getRuntime().exec" --include="*.java" -n
grep -r "SELECT.*FROM.*WHERE.*\+" --include="*.java" -n  # SQL injection

# Performance patterns
grep -r "\.parallelStream()" --include="*.java" -n
grep -r "for.*for.*for" --include="*.java" -n  # Nested loops
grep -r "saveAll(" --include="*.java" -n  # Batch operations

# Concurrency patterns
grep -r "synchronized" --include="*.java" -n
grep -r "new Thread(" --include="*.java" -n
grep -r "Executors\.new" --include="*.java" -n

# Configuration patterns
grep -r "timeout.*[0-9]" --include="*.yml" --include="*.properties" -n
grep -r "pool.*size" --include="*.yml" --include="*.properties" -n
```

**Output**: `hotspots.json` - List of files requiring deep analysis

```json
{
  "security": [
    {"file": "UserController.java", "line": 45, "pattern": "password=", "priority": "CRITICAL"}
  ],
  "performance": [
    {"file": "EventService.java", "line": 123, "pattern": "saveAll(", "priority": "HIGH"}
  ],
  "concurrency": [
    {"file": "CRMService.java", "line": 374, "pattern": "new Thread(", "priority": "HIGH"}
  ]
}
```

---

### Phase 3: Parallel Agent Execution

**Goal**: Launch specialized agents concurrently, each with 30-50k token budget

**Agent Coordination**:

Each agent receives:
1. **Manifest** - Structural context
2. **Hotspots** - Prioritized file list for their domain
3. **Layer Assignment** - Specific architectural layer to analyze
4. **Context** - Cross-references to other layers (e.g., service → repository calls)
5. **Completeness Rules** - Mandatory enforcement instructions (Phase 0)

**Validation Requirements**:
- Agent output MUST include `analysis_metadata` with `declared_count` and `actual_count`
- Output MUST include `validation` block confirming completeness
- Orchestrator MUST validate `declared_count === actual_count` before accepting results
- Any summarization detected = output REJECTED, agent must re-run

#### Progressive Writing Strategy (v2.4)

**For Large Codebases (>100K LOC)**:

Instead of returning all findings in response, agents write progressively to disk:

```bash
# Each agent writes to its own file
Security Agent → security_findings.md
Performance Agent → performance_findings.md
Concurrency Agent → concurrency_findings.md
Architecture Agent → architecture_findings.md
```

**Agent Pattern**:
1. Initialize output file with **Quick Reference Table** header
2. Analyze files in batches
3. Write findings to file every 50 issues
4. **Clear findings from context** after writing
5. Continue analysis with freed context
6. Apply **v2.4 output strategy**: ALL CRITICAL + ALL HIGH detailed, 5 MEDIUM + 5 LOW samples, rest in Quick Reference Table
7. Return SUMMARY only (not full findings)

**Agent Final Response** (Summary Only):
```json
{
  "agent": "Security Agent",
  "status": "completed",
  "output_file": "security_findings.md",
  "findings_found": 250,
  "findings_documented": 250,
  "output_strategy": "v2.4",
  "breakdown": {
    "CRITICAL": {"found": 8, "detailed": 8, "in_table": 0},
    "HIGH": {"found": 42, "detailed": 42, "in_table": 0},
    "MEDIUM": {"found": 120, "detailed": 5, "in_table": 115},
    "LOW": {"found": 80, "detailed": 5, "in_table": 75}
  },
  "context_usage": "48%"
}
```

**v2.4 Benefits**:
- No output token overflow (32K limit avoided)
- Constant context usage (~50KB)
- **ALL findings preserved** (100% documented)
- **Quick Reference Tables** for instant navigation
- ALL CRITICAL/HIGH issues detailed (not sampled)
- Scalable to 1M+ LOC codebases

**Example Agent Prompt**:

```markdown
# SECURITY AGENT TASK

## Context (from manifest.json)
- Project: sport-gestione-licenze-service
- Framework: Spring Boot 3.5.5
- Total Files: 845
- Your Focus: Controller + Integration layers (66 files)

## Hotspots (from pattern scan)
You MUST analyze these files in detail:
1. UserController.java:45 - password= pattern found
2. AuthService.java:89 - exec() pattern found
3. DatabaseUtil.java:234 - SQL concatenation found

## Your Mission
Analyze the controller and integration layers for:
- Input validation gaps
- Authentication/authorization bypass
- SQL injection vectors
- Hardcoded secrets
- Insecure deserialization

## Output Format
Return JSON object with metadata and findings array:
{
  "analysis_metadata": {
    "agent_type": "security",
    "declared_count": XX,
    "actual_count": XX,
    "completeness": "100%",
    "status": "COMPLETE"
  },
  "findings": [
    {
      "id": "SEC-001",
      "type": "SECURITY",
      "severity": "CRITICAL",
      "category": "SQL_INJECTION",
      "file": "path/to/file.java",
      "line": 123,
      "evidence": "actual code snippet",
      "description": "SQL query built with string concatenation",
      "impact": "Attacker can execute arbitrary SQL",
      "recommendation": "Use PreparedStatement with parameterized queries"
    }
  ],
  "validation": {
    "id_sequence_valid": true,
    "no_duplicates": true,
    "all_have_evidence": true,
    "all_have_recommendations": true,
    "counts_match": true
  }
}

## Constraints
- Token budget: 40,000
- Priority: CRITICAL > HIGH > MEDIUM > LOW
- **MANDATORY**: Follow COMPLETENESS ENFORCEMENT - document ALL findings individually
- **NO SUMMARIZATION**: Never group findings (e.g., "8 SQL injections found")
```

---

### Phase 4: Agent Catalog

#### 1. Security Agent

**Focus**: Authentication, authorization, input validation, secrets, cryptography

**Language Plugins**:
- **Java**: Spring Security, JWT, OAuth2, SQL injection, XXE, deserialization
- **Python**: Django auth, Flask-Security, SQL injection, command injection
- **JavaScript**: Passport.js, XSS, CSRF, prototype pollution

**Patterns**:
```bash
# Java
@PreAuthorize|@Secured|@RolesAllowed
password|secret|apikey|token
PreparedStatement|createQuery
MessageDigest|Cipher|SecureRandom

# Python
authenticate|login_required|permission_required
hashlib|bcrypt|Fernet
cursor.execute|raw|extra

# JavaScript
passport|jwt|bcrypt
eval|innerHTML|dangerouslySetInnerHTML
req.query|req.params|req.body
```

**Output**: `findings-security.json`

---

#### 2. Performance Agent

**Focus**: Database queries, N+1 problems, loops, caching, algorithm complexity

**Language Plugins**:
- **Java**: Hibernate N+1, missing indexes, transaction boundaries, connection pools
- **Python**: Django ORM select_related, SQLAlchemy lazy loading
- **JavaScript**: Mongoose populate, missing indices

**Patterns**:
```bash
# Java
@Query|findAll|findBy|saveAll
@Transactional|@Lock
for.*for.*for|while.*while
@Cacheable|@CacheEvict

# Python
.filter(|.all(|.get(
select_related|prefetch_related
for.*for.*for

# JavaScript
.find(|.findOne(|.insertMany(
populate|lean
for.*for.*of
```

**Special Analysis**:
- Calculate cyclomatic complexity for methods with nested loops
- Identify O(n²) or worse algorithms
- Check batch size configurations
- Verify connection pool settings

**Output**: `findings-performance.json`

---

#### 3. Concurrency Agent

**Focus**: Thread safety, race conditions, deadlocks, shared state

**Language Plugins**:
- **Java**: synchronized, volatile, Atomic, ConcurrentHashMap, ExecutorService
- **Python**: threading, multiprocessing, asyncio, locks
- **JavaScript**: async/await, Promise.all, worker threads

**Patterns**:
```bash
# Java
synchronized|volatile|Atomic|Concurrent
ExecutorService|ThreadPool|parallelStream
wait\(|notify\(|join\(

# Python
threading\.|multiprocessing\.|asyncio\.
Lock|Semaphore|Queue
global.*=

# JavaScript
async|await|Promise|Worker
setTimeout|setInterval|setImmediate
```

**Critical Checks**:
- Shared mutable state modified by multiple threads
- ExecutorService lifecycle management (shutdown)
- Parallel stream operations on non-thread-safe collections
- Missing synchronization on shared resources

**Output**: `findings-concurrency.json`

---

#### 4. JPA/Hibernate Agent (Java-specific)

**Focus**: Entity optimization, lazy loading, batch configuration, caching

**Patterns**:
```bash
@Entity|@Table|@OneToMany|@ManyToOne
fetch.*LAZY|fetch.*EAGER
@BatchSize|@Fetch
cascade.*ALL|orphanRemoval
```

**Deep Analysis**:
1. Count total entities
2. Identify lazy relationships WITHOUT @BatchSize
3. Check application.yml for:
   - hibernate.jdbc.batch_size
   - hibernate.order_inserts
   - hibernate.default_batch_fetch_size
4. Find saveAll() operations without batch config
5. Check second-level cache configuration

**Output**: `findings-jpa.json`

---

#### 5. Resilience Agent

**Focus**: Timeouts, retries, circuit breakers, bulkheads, fallbacks

**Language Plugins**:
- **Java**: Resilience4j, Feign, Hystrix, timeout configs
- **Python**: tenacity, requests timeout, celery retries
- **JavaScript**: axios timeout, retry-axios, opossum (circuit breaker)

**Patterns**:
```bash
# Java
@CircuitBreaker|@Retry|@Bulkhead|@RateLimiter
connectTimeout|readTimeout
@FeignClient

# Python
@retry|@timeout
requests.get.*timeout=
celery.task.*retry

# JavaScript
timeout:|retry:|circuitBreaker:
axios.create|fetch\(
```

**Critical Checks**:
- Excessive timeouts (>30s)
- Missing circuit breakers on external calls
- Missing retries with exponential backoff
- Thread pool sizing for async operations

**Output**: `findings-resilience.json`

---

#### 6. Architecture Agent

**Focus**: Dependency violations, circular dependencies, layer boundaries, coupling

**Analysis**:
1. Build dependency graph
2. Detect circular dependencies
3. Identify layer violations (controller → repository direct call)
4. Calculate coupling metrics
5. Find god classes (>1000 LOC or >50 methods)

**Output**: `findings-architecture.json`

---

### Phase 5: Result Assembly & Deduplication

**Goal**: Merge findings from all agents, eliminate duplicates, prioritize

**Deduplication Algorithm**:

```python
def canonical_hash(finding):
    """Generate unique hash for finding"""
    return hashlib.sha256(
        f"{finding['file']}:{finding['line']}:{finding['category']}"
        .encode()
    ).hexdigest()[:16]

def deduplicate(all_findings):
    seen = {}
    unique = []

    for finding in all_findings:
        hash = canonical_hash(finding)

        if hash in seen:
            # Keep higher severity
            if SEVERITY[finding['severity']] > SEVERITY[seen[hash]['severity']]:
                unique.remove(seen[hash])
                unique.append(finding)
                seen[hash] = finding
        else:
            unique.append(finding)
            seen[hash] = finding

    return unique

SEVERITY = {
    'CRITICAL': 4,
    'HIGH': 3,
    'MEDIUM': 2,
    'LOW': 1
}
```

**Output**: `findings-deduplicated.json`

---

### Phase 5.5: Agent Output Validation

**Goal**: Ensure every agent output meets completeness requirements before assembly

**Validation Script** (run after each agent completes):

```python
def validate_agent_output(output_json, agent_name):
    """Validate agent output for completeness"""

    # Check metadata exists
    if 'analysis_metadata' not in output_json:
        raise ValueError(f"{agent_name}: Missing analysis_metadata")

    metadata = output_json['analysis_metadata']
    findings = output_json['findings']

    # Check counts match
    declared = metadata.get('declared_count', 0)
    actual = len(findings)

    if actual < declared:
        raise ValueError(
            f"{agent_name}: INCOMPLETE - Declared {declared} findings "
            f"but only {actual} extracted. Missing {declared - actual} findings!"
        )

    # Check ID sequence
    ids = [f['id'] for f in findings]
    prefix = agent_name[:3].upper()
    expected_ids = [f"{prefix}-{i:03d}" for i in range(1, actual + 1)]

    missing_ids = set(expected_ids) - set(ids)
    if missing_ids:
        raise ValueError(
            f"{agent_name}: ID sequence broken. Missing IDs: {missing_ids}"
        )

    # Check for placeholders/summarization
    for finding in findings:
        desc = finding.get('description', '')
        if any(phrase in desc.lower() for phrase in ['...', 'etc', 'and others', 'similar']):
            raise ValueError(
                f"{agent_name}: Summarization detected in {finding['id']}: {desc}"
            )

        # Check all required fields
        required = ['id', 'type', 'severity', 'file', 'line', 'evidence', 'description']
        missing = [field for field in required if field not in finding or not finding[field]]
        if missing:
            raise ValueError(
                f"{agent_name}: {finding['id']} missing required fields: {missing}"
            )

    # Check validation block
    validation = output_json.get('validation', {})
    if not all(validation.values()):
        failed = [k for k, v in validation.items() if not v]
        raise ValueError(
            f"{agent_name}: Validation failed for: {failed}"
        )

    print(f"✅ {agent_name}: Validation PASSED - {actual} findings documented")
    return True
```

**Integration**:
- Run validation immediately after each agent completes
- Log validation failures for debugging
- Reject incomplete outputs and re-run agent with stricter instructions
- Only proceed to deduplication after ALL agents pass validation

---

### Phase 6: Report Generation (v2.4)

**Goal**: Transform JSON findings into professional markdown report with complete coverage

**v2.4 Report Structure**:

```markdown
================================================================================
                           CODE REVIEW REPORT
                     [Project Name from manifest]
================================================================================
Report ID: [YYYY-MM-DD-XXXX]
Data Analisi: [YYYY-MM-DD]
Tipo: Performance & Security Code Review
Analisi: Completa 100%
Framework Version: 2.4

## 1. EXECUTIVE SUMMARY

### Environment
- **Linguaggi**: [from manifest]
- **Framework**: [from manifest]
- **Database**: [from manifest]
- **Total Files**: [from manifest]
- **Total LOC**: [from manifest]

### Findings Overview (v2.4 Complete Coverage)
- **CRITICAL**: [count] issues requiring immediate attention [ALL detailed below]
- **HIGH**: [count] issues requiring near-term resolution [ALL detailed below]
- **MEDIUM**: [count] issues for backlog [5 samples detailed, rest in Quick Reference]
- **LOW**: [count] minor improvements [5 samples detailed, rest in Quick Reference]
- **TOTAL**: [count] findings (100% documented)

## 2. PROJECT STRUCTURE
[From manifest - architecture section]

## 3. CRITICAL & HIGH PRIORITY ISSUES (ALL Detailed)

**v2.4 Strategy**: ALL CRITICAL and HIGH findings documented in full detail (5 lines each)

Context-optimized format: 5 lines per issue, focus on WHAT and WHY

### [DOMAIN]-001: Title
**File**: `path/to/file:line`
**Severity**: CRITICAL | HIGH
**Problem**: [1-2 line factual description]
**Impact**: [concrete impact - minimal]
**Fix**: [1 line hint]

---

[Repeat for ALL CRITICAL findings]
[Repeat for ALL HIGH findings]

## 4. MEDIUM PRIORITY ISSUES (Representative Samples)

**v2.4 Strategy**: 5 detailed samples + Quick Reference Table for all remaining

[5 detailed MEDIUM findings in same format as above]

---

### Quick Reference: All Remaining MEDIUM Issues

| ID | Severity | Category | File:Line | Brief Description |
|----|----------|----------|-----------|-------------------|
| [DOMAIN]-XXX | MEDIUM | [CATEGORY] | path/file.ext:123 | One-line description |
| [... all remaining MEDIUM issues ...] |

## 5. LOW PRIORITY ISSUES (Representative Samples)

**v2.4 Strategy**: 5 detailed samples + Quick Reference Table for all remaining

[5 detailed LOW findings in same format as above]

---

### Quick Reference: All Remaining LOW Issues

| ID | Severity | Category | File:Line | Brief Description |
|----|----------|----------|-----------|-------------------|
| [DOMAIN]-XXX | LOW | [CATEGORY] | path/file.ext:123 | One-line description |
| [... all remaining LOW issues ...] |

## 6. FINDINGS BY DOMAIN (Quick Navigation Index)

### Security Findings Index
See: `security_findings.md` for complete Quick Reference Table

**Summary**:
- CRITICAL: X (all detailed in Section 3)
- HIGH: Y (all detailed in Section 3)
- MEDIUM: Z (5 detailed in Section 4, rest in Quick Reference)
- LOW: W (5 detailed in Section 5, rest in Quick Reference)

### Performance Findings Index
See: `performance_findings.md` for complete Quick Reference Table

[Similar breakdown for each domain]

## 7. STATISTICS

### By Severity
- CRITICAL: X (100% detailed)
- HIGH: Y (100% detailed)
- MEDIUM: Z (5 detailed + [Z-5] in Quick Reference)
- LOW: W (5 detailed + [W-5] in Quick Reference)

### By Category
- Security: X findings
- Performance: Y findings
- Concurrency: Z findings
- Architecture: W findings
- [see domain-specific files for Quick Reference Tables]

### By Layer
- Controller: X findings
- Service: Y findings
- Repository: Z findings
- Integration: W findings

## 8. RECOMMENDATIONS

### Immediate Actions (This Sprint)
1. [Ordered by severity and impact - focus on CRITICAL]

### Near-Term (Next 2 Sprints)
1. [Ordered by value/effort ratio - focus on HIGH]

### Long-Term (Architectural)
1. [Strategic improvements - MEDIUM/LOW with high impact]

## 9. APPENDIX

### Analysis Methodology (v2.4)
- Framework Version: 2.4
- Agents Used: [list]
- Total Files Analyzed: [number]
- Output Strategy: ALL CRITICAL + ALL HIGH detailed, 5 MEDIUM + 5 LOW samples
- Quick Reference Tables: YES (all findings indexed)
- Analysis Duration: [hours]
- Token Budget Used: [number]

### References
- [Link to OWASP guidelines if security issues]
- [Link to performance benchmarks]
- [Link to concurrency best practices]

### Navigation Guide
- For CRITICAL/HIGH findings: See Section 3 (all detailed)
- For MEDIUM findings: See Section 4 (samples) + domain Quick Reference Tables
- For LOW findings: See Section 5 (samples) + domain Quick Reference Tables
- For complete domain analysis: See `{domain}_findings.md` files
```

---

## TACTICAL APPROACHES EXPLAINED

### 1. Semantic Segmentation

**Problem**: Splitting a 100k LOC codebase by file count loses architectural context

**Solution**: Segment by architectural layer

```
Agent 1: Controllers (42 files, 8k LOC) + Integration (24 files, 6k LOC)
Agent 2: Services (156 files, 45k LOC) - split into 3 sub-agents
Agent 3: DAO/Repository (189 files, 32k LOC) - split into 3 sub-agents
Agent 4: Utilities (38 files, 5k LOC)
```

**Benefits**:
- Each agent understands its layer's purpose
- Cross-layer references maintained via manifest
- Natural separation of concerns

---

### 2. Context Injection

**Problem**: Agent analyzing ServiceImpl.java doesn't know what Controller.java expects

**Solution**: Inject minimal context from other layers

```json
{
  "file_to_analyze": "UserServiceImpl.java",
  "context": {
    "callers": [
      {"file": "UserController.java", "method": "createUser", "line": 45},
      {"file": "AdminController.java", "method": "bulkCreateUsers", "line": 123}
    ],
    "callees": [
      {"file": "UserRepository.java", "method": "save"},
      {"file": "EmailService.java", "method": "sendWelcomeEmail"}
    ],
    "related_entities": ["User.java", "UserRole.java"]
  }
}
```

**Benefits**:
- Agent understands usage patterns
- Can identify mismatches (e.g., controller expects validation, service doesn't provide)
- Token budget: only +500 tokens for critical context

---

### 3. Pattern-Based Scanning (Grep First)

**Problem**: Reading 845 files completely exceeds token budget

**Solution**: Grep patterns identify which files need deep analysis

```bash
# Find all Feign clients
grep -r "@FeignClient" --include="*.java" -l > feign-clients.txt
# Output: 15 files

# Pass only these 15 files to Resilience Agent
```

**Benefits**:
- 845 files → 15 files for deep analysis
- Agents focus on hotspots
- 90% reduction in token usage

---

### 4. Tiered Analysis

**Problem**: Not all code needs same depth of analysis

**Solution**: Three-tier approach

**Tier 1 - Quick Scan (All files, pattern matching)**:
- Grep for anti-patterns
- Complexity metrics (LOC, cyclomatic)
- Dependency violations
- **Time**: 5 minutes, **Token**: 0

**Tier 2 - Standard Analysis (50% of files)**:
- Agent reads file
- Checks common issues
- Uses language plugin patterns
- **Time**: 30 minutes, **Token**: 30k per agent

**Tier 3 - Deep Dive (10% of files - hotspots)**:
- Line-by-line analysis
- Cross-reference verification
- Business logic validation
- **Time**: 2 hours, **Token**: 50k per agent

---

### 5. Language Plugins

**Problem**: Framework needs to work for Java, Python, JavaScript, Go, etc.

**Solution**: Core patterns + language-specific plugins

**Core Framework** (language-agnostic):
- Discovery workflow
- Manifest generation
- Agent orchestration
- Result assembly

**Language Plugins** (drop-in modules):
```
plugins/
├── java/
│   ├── patterns.yaml (Spring, Hibernate, Feign patterns)
│   ├── security.py (SQL injection, XXE, deserialization)
│   └── performance.py (N+1, transaction boundaries)
├── python/
│   ├── patterns.yaml (Django, Flask, SQLAlchemy)
│   ├── security.py (SQL injection, command injection)
│   └── performance.py (ORM optimization)
└── javascript/
    ├── patterns.yaml (Express, React, Mongoose)
    ├── security.py (XSS, CSRF, prototype pollution)
    └── performance.py (async/await, event loop blocking)
```

**Auto-detection**:
```python
def detect_language_and_load_plugin(manifest):
    if "pom.xml" in manifest['files']:
        return load_plugin("java")
    elif "requirements.txt" in manifest['files']:
        return load_plugin("python")
    elif "package.json" in manifest['files']:
        return load_plugin("javascript")
```

---

### 6. Size-Based Routing

**Problem**: Files vary from 50 LOC to 5000 LOC

**Solution**: Different strategies for different sizes

```python
def route_file(file_path, loc):
    if loc < 200:
        # Small file - read entirely
        return "READ_FULL"

    elif loc < 1000:
        # Medium file - read with context window
        return "READ_WINDOWED"

    elif loc < 3000:
        # Large file - read critical sections only
        sections = extract_critical_sections(file_path)
        return {"strategy": "READ_SECTIONS", "sections": sections}

    else:
        # Huge file - red flag, needs refactoring
        return {"strategy": "REPORT_GOD_CLASS", "recommendation": "Split file"}
```

---

## LANGUAGE-SPECIFIC CONFIGURATIONS

### Java (Spring Boot)

**Discovery Patterns**:
```bash
# Framework detection
test -f pom.xml && echo "Maven" || echo "Gradle"
grep -q "spring-boot-starter" pom.xml && echo "Spring Boot"

# Layer detection
find src/main/java -type d -name "controller" -o -name "service" -o -name "repository"
```

**Critical Files**:
- `application.yml` / `application.properties`
- `pom.xml` / `build.gradle`
- `@SpringBootApplication` class

**Agent Priority**:
1. JPA/Hibernate Agent (entities, repositories, queries)
2. Performance Agent (batch operations, caching, transactions)
3. Resilience Agent (Feign, RestTemplate, timeouts)
4. Security Agent (Spring Security, input validation, SQL injection)
5. Concurrency Agent (parallel streams, thread pools, synchronized)

---

### Python (Django/Flask)

**Discovery Patterns**:
```bash
# Framework detection
test -f manage.py && echo "Django"
grep -q "Flask" requirements.txt && echo "Flask"

# Layer detection
find . -name "views.py" -o -name "models.py" -o -name "serializers.py"
```

**Critical Files**:
- `settings.py` / `config.py`
- `requirements.txt`
- `models.py` (all ORM models)

**Agent Priority**:
1. Security Agent (SQL injection, XSS, CSRF, authentication)
2. Performance Agent (ORM N+1, select_related, database indexes)
3. ORM Agent (Django ORM / SQLAlchemy optimization)
4. Concurrency Agent (async views, Celery tasks, threading)

---

### JavaScript (Node.js / Express)

**Discovery Patterns**:
```bash
# Framework detection
test -f package.json && echo "Node.js"
grep -q "express" package.json && echo "Express"
grep -q "react" package.json && echo "React"

# Layer detection
find . -name "routes" -type d -o -name "controllers" -type d -o -name "models" -type d
```

**Critical Files**:
- `package.json`
- `app.js` / `index.js` / `server.js`
- `.env` / `config/`

**Agent Priority**:
1. Security Agent (XSS, CSRF, prototype pollution, input validation)
2. Performance Agent (async/await, Promise handling, database queries)
3. Concurrency Agent (event loop blocking, CPU-intensive operations)
4. Architecture Agent (callback hell, promise chains, error handling)

---

## EXAMPLE ORCHESTRATION SCRIPT

```python
#!/usr/bin/env python3
"""
Universal Code Review Orchestrator
Works with any language/framework
"""

import json
import subprocess
import concurrent.futures
from pathlib import Path

class CodeReviewOrchestrator:
    def __init__(self, repo_path):
        self.repo_path = Path(repo_path)
        self.manifest = {}
        self.findings = []

    def run(self):
        """Main orchestration workflow"""
        print("🔍 Phase 1: Discovery")
        self.manifest = self.discover()
        self.save_json("manifest.json", self.manifest)

        print("🎯 Phase 2: Pattern Scanning")
        hotspots = self.scan_patterns()
        self.save_json("hotspots.json", hotspots)

        print("🤖 Phase 3: Parallel Agent Execution")
        findings = self.run_agents_parallel(hotspots)

        print("🔗 Phase 4: Deduplication")
        unique_findings = self.deduplicate(findings)
        self.save_json("findings-final.json", unique_findings)

        print("📄 Phase 5: Report Generation")
        self.generate_report(unique_findings)

        print(f"✅ Complete! {len(unique_findings)} findings documented")

    def discover(self):
        """Phase 1: Discover project structure"""
        manifest = {
            "project_name": self.repo_path.name,
            "languages": self.detect_languages(),
            "frameworks": self.detect_frameworks(),
            "total_files": self.count_files(),
            "total_loc": self.count_loc(),
            "architecture": self.detect_architecture()
        }
        return manifest

    def detect_languages(self):
        """Auto-detect programming languages"""
        extensions = {
            ".java": "Java",
            ".py": "Python",
            ".js": "JavaScript",
            ".ts": "TypeScript",
            ".go": "Go",
            ".rb": "Ruby",
            ".php": "PHP"
        }

        languages = set()
        for ext, lang in extensions.items():
            if list(self.repo_path.rglob(f"*{ext}")):
                languages.add(lang)

        return sorted(languages)

    def detect_frameworks(self):
        """Auto-detect frameworks"""
        frameworks = []

        # Java
        if (self.repo_path / "pom.xml").exists():
            frameworks.append("Maven")
            pom_content = (self.repo_path / "pom.xml").read_text()
            if "spring-boot" in pom_content:
                frameworks.append("Spring Boot")
            if "hibernate" in pom_content:
                frameworks.append("Hibernate")

        # Python
        if (self.repo_path / "requirements.txt").exists():
            req_content = (self.repo_path / "requirements.txt").read_text()
            if "Django" in req_content:
                frameworks.append("Django")
            if "Flask" in req_content:
                frameworks.append("Flask")

        # JavaScript
        if (self.repo_path / "package.json").exists():
            pkg_content = json.loads((self.repo_path / "package.json").read_text())
            deps = {**pkg_content.get("dependencies", {}), **pkg_content.get("devDependencies", {})}
            if "express" in deps:
                frameworks.append("Express")
            if "react" in deps:
                frameworks.append("React")

        return frameworks

    def scan_patterns(self):
        """Phase 2: Quick pattern scanning with grep"""
        hotspots = {
            "security": [],
            "performance": [],
            "concurrency": []
        }

        # Load language-specific patterns
        plugin = self.load_language_plugin(self.manifest['languages'][0])

        for category, patterns in plugin['patterns'].items():
            for pattern in patterns:
                results = self.grep(pattern)
                hotspots[category].extend(results)

        return hotspots

    def grep(self, pattern):
        """Execute grep and parse results"""
        result = subprocess.run(
            ["grep", "-r", "-n", pattern, str(self.repo_path)],
            capture_output=True,
            text=True
        )

        matches = []
        for line in result.stdout.split("\n"):
            if ":" in line:
                file_path, line_num, *_ = line.split(":", 2)
                matches.append({
                    "file": file_path,
                    "line": int(line_num),
                    "pattern": pattern
                })

        return matches

    def run_agents_parallel(self, hotspots):
        """Phase 3: Run agents in parallel"""
        agents = [
            ("security", self.run_security_agent),
            ("performance", self.run_performance_agent),
            ("concurrency", self.run_concurrency_agent),
            ("jpa", self.run_jpa_agent),
            ("resilience", self.run_resilience_agent)
        ]

        all_findings = []

        with concurrent.futures.ThreadPoolExecutor(max_workers=5) as executor:
            futures = {
                executor.submit(agent_fn, hotspots): name
                for name, agent_fn in agents
            }

            for future in concurrent.futures.as_completed(futures):
                agent_name = futures[future]
                findings = future.result()
                all_findings.extend(findings)
                print(f"✅ {agent_name} complete: {len(findings)} findings")

        return all_findings

    def run_security_agent(self, hotspots):
        """Launch security agent"""
        # Call Claude Code agent with specialized prompt
        prompt = self.build_agent_prompt("security", hotspots['security'])
        findings = self.call_agent(prompt)
        return findings

    def deduplicate(self, findings):
        """Phase 4: Remove duplicates"""
        import hashlib

        seen = {}
        unique = []

        for finding in findings:
            key = f"{finding['file']}:{finding['line']}:{finding['category']}"
            hash = hashlib.sha256(key.encode()).hexdigest()[:16]

            if hash not in seen:
                unique.append(finding)
                seen[hash] = finding

        return sorted(unique, key=lambda x: (
            {"CRITICAL": 0, "HIGH": 1, "MEDIUM": 2, "LOW": 3}[x['severity']],
            x['file'],
            x['line']
        ))

    def generate_report(self, findings):
        """Phase 5: Generate markdown report"""
        report = self.build_report_markdown(findings)
        (self.repo_path / "CODE_REVIEW_REPORT.md").write_text(report)

if __name__ == "__main__":
    import sys

    if len(sys.argv) < 2:
        print("Usage: python orchestrator.py <repo_path>")
        sys.exit(1)

    orchestrator = CodeReviewOrchestrator(sys.argv[1])
    orchestrator.run()
```

---

## TOKEN BUDGET MANAGEMENT

### Budget Allocation (Total: 200k tokens)

```
Discovery Phase:        5,000 tokens (2.5%)
Pattern Scanning:       2,000 tokens (1%)
Agent Execution:      150,000 tokens (75%)
  ├─ Security:         30,000 tokens
  ├─ Performance:      35,000 tokens
  ├─ Concurrency:      25,000 tokens
  ├─ JPA/ORM:          30,000 tokens
  └─ Resilience:       30,000 tokens
Assembly/Dedupe:       8,000 tokens (4%)
Report Generation:    35,000 tokens (17.5%)
```

### Overflow Strategy

If a layer exceeds token budget:

1. **Split by file count**: Divide 156 service files into 3 batches
2. **Run sequentially**: Agent 1 → merge → Agent 2 → merge → Agent 3
3. **Context injection**: Pass previous findings to next batch
4. **Hash tracking**: Maintain deduplication hash set across batches

---

## VALIDATION CHECKLIST

Before finalizing report:

### Completeness Enforcement
- [ ] All agents passed Phase 5.5 validation (declared_count === actual_count)
- [ ] No summarization detected in any agent output
- [ ] All finding IDs sequential without gaps
- [ ] All findings have complete required fields

### Standard Validation
- [ ] All files in manifest analyzed or explicitly skipped
- [ ] Every finding has file:line reference
- [ ] Every finding has code evidence
- [ ] Severity levels justified by standard (OWASP, CWE, etc.)
- [ ] No invented technologies (only detected ones)
- [ ] Statistics match finding counts
- [ ] Cross-references verified (if service calls repository, both exist)
- [ ] Quick wins section has concrete configurations
- [ ] Recommendations prioritized by impact/effort
- [ ] Report generated in <2 hours total time

---

## EXTENDING THE FRAMEWORK

### Adding New Language Support

1. Create `plugins/{language}/` directory
2. Add `patterns.yaml`:
```yaml
security:
  - "password.*="
  - "eval\("
  - "exec\("

performance:
  - "sleep\("
  - "for.*for.*for"

concurrency:
  - "threading\."
  - "multiprocessing\."
```

3. Add `{category}.py` with language-specific logic
4. Update `orchestrator.py` to load plugin

### Adding New Agent Type

1. Define agent mission and scope
2. Create prompt template in `AGENT-PROMPTS.md`
3. Add to orchestrator agent list
4. Define output schema
5. Add to report generation

---

## CONCLUSION

This framework provides:

1. **Scalability**: Handle 10k to 100k+ LOC
2. **Adaptability**: Works with any language
3. **Completeness**: Line-by-line analysis without losing context
4. **Efficiency**: Parallel execution + pattern scanning
5. **Accuracy**: Factual findings with evidence
6. **Actionability**: Prioritized recommendations with quick wins

**Next Steps**: See `START-HERE.md` for getting started and `EXAMPLES.md` for hands-on examples
