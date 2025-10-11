# CLAUDE CODE ANALYSIS FRAMEWORK
## Universal Deep-Dive Code Review System

**Version**: 2.0
**Last Updated**: 2025-10-11
**Purpose**: Programmatic, scalable code analysis framework that works with any repository size and programming language

---

## OVERVIEW

This framework enables **line-by-line code analysis** of repositories of any size by using:

1. **Semantic Segmentation** - Divide by architectural layers, not arbitrary chunks
2. **Specialized Agents** - Parallel execution of domain-specific analyzers
3. **Context Injection** - Maintain system awareness across agent boundaries
4. **Pattern-Based Scanning** - Identify hotspots before deep analysis
5. **Tiered Analysis** - Adaptive depth based on risk and complexity
6. **Language Plugins** - Core patterns + language-specific extensions
7. **Hash-Based Deduplication** - Canonical identification of duplicate findings
8. **Dependency Graph Awareness** - Understand system-wide relationships
9. **Cross-Cutting Concerns** - Agents that analyze multiple layers
10. **Size-Based Routing** - Adaptive strategies for different file sizes

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
Return JSON array of findings:
[
  {
    "id": "SEC-CRIT-001",
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
]

## Constraints
- Token budget: 40,000
- Max findings: 100
- Priority: CRITICAL > HIGH > MEDIUM > LOW
- If you find >100 issues, return only CRITICAL and HIGH
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

### Phase 6: Report Generation

**Goal**: Transform JSON findings into professional markdown report

**Report Structure**:

```markdown
================================================================================
                           CODE REVIEW REPORT
                     [Project Name from manifest]
================================================================================
Report ID: [YYYY-MM-DD-XXXX]
Data Analisi: [YYYY-MM-DD]
Tipo: Performance & Security Code Review
Analisi: Completa 100%

## 1. EXECUTIVE SUMMARY

### Environment
- **Linguaggi**: [from manifest]
- **Framework**: [from manifest]
- **Database**: [from manifest]
- **Total Files**: [from manifest]
- **Total LOC**: [from manifest]

### Findings Overview
- **CRITICAL**: [count] issues requiring immediate attention
- **HIGH**: [count] issues requiring near-term resolution
- **MEDIUM**: [count] issues for backlog
- **LOW**: [count] minor improvements
- **TOTAL**: [count] findings

## 2. PROJECT STRUCTURE
[From manifest - architecture section]

## 3. CRITICAL ISSUES (Risoluzione Immediata)

### [CRIT-001] Title
**File**: `path/to/file:line`
**Type**: SECURITY | PERFORMANCE | CONCURRENCY
**Category**: [specific category]

**Code**:
```java
[actual code snippet]
```

**Problem**: [factual description]

**Impact**: [concrete impact]

**Recommendation**: [actionable fix]

---

[Repeat for all CRITICAL]

## 4. HIGH PRIORITY ISSUES
[Same format]

## 5. MEDIUM PRIORITY ISSUES
[Same format - may be summarized if >50]

## 6. LOW PRIORITY ISSUES
[Same format - may be summarized if >100]

## 7. QUICK WINS (1-2 Hour Fixes)

### Configuration Changes
```yaml
# Add to application.yml
[specific configs with values]
```

### Code Patterns to Replace
[Specific before/after examples]

## 8. STATISTICS

### By Severity
- CRITICAL: X
- HIGH: Y
- MEDIUM: Z
- LOW: W

### By Category
- Security: X
- Performance: Y
- Concurrency: Z
- Architecture: W

### By Layer
- Controller: X
- Service: Y
- Repository: Z
- Integration: W

## 9. RECOMMENDATIONS

### Immediate Actions (This Sprint)
1. [Ordered by severity and impact]

### Near-Term (Next 2 Sprints)
1. [Ordered by value/effort ratio]

### Long-Term (Architectural)
1. [Strategic improvements]

## 10. APPENDIX

### Analysis Methodology
- Agents Used: [list]
- Total Files Analyzed: [number]
- Analysis Duration: [hours]
- Token Budget Used: [number]

### References
- [Link to OWASP guidelines if security issues]
- [Link to performance benchmarks]
- [Link to concurrency best practices]
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

**Next Steps**: See `QUICK-START.md` for hands-on examples
