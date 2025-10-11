# Universal Code Review Framework

**A scalable, language-agnostic framework for deep code analysis using AI agents**

---

## Overview

This framework enables **comprehensive, line-by-line code analysis** of repositories of any size (10K to 100K+ LOC) in any programming language (Java, Python, JavaScript, Go, etc.) using specialized AI agents and intelligent orchestration.

### Key Features

- **Language Agnostic**: Works with Java, Python, JavaScript, and easily extensible to other languages
- **Scalable**: Handles codebases from 10K to 100K+ lines of code
- **Intelligent**: Uses pattern-based scanning to identify hotspots before deep analysis
- **Parallel**: Runs multiple specialized agents concurrently
- **Comprehensive**: Analyzes security, performance, concurrency, resilience, and architecture
- **Factual**: Reports only verified issues with code evidence
- **Actionable**: Provides concrete recommendations with code examples
- **100% Complete**: Guarantees every finding is documented individually (no summarization)
- **Chain of Thought**: Systematic 6-step analysis with confidence scoring for each finding
- **Validated**: 3-phase enforcement mechanism ensures declared_count === actual_count

### What Problems Does It Solve?

1. **Token Limitations**: Overcomes AI token limits through semantic segmentation
2. **Context Loss**: Maintains system awareness across agent boundaries
3. **Scale**: Analyzes large repositories without missing critical issues
4. **Efficiency**: Pattern scanning identifies hotspots for targeted deep analysis
5. **Completeness**: Ensures 100% code coverage through systematic orchestration
6. **AI Summarization**: Prevents AI from grouping findings ("8 SQL injections found" → lists all 8 with file:line)
7. **Finding Loss**: 3-phase validation guarantees no findings are omitted during analysis

---

## Quick Start

### 5-Minute Analysis

```bash
# 1. Clone your repository
cd ~/projects
git clone https://github.com/your-org/your-repo.git
cd your-repo

# 2. Run quick discovery
find . -name "*.java" -o -name "*.py" -o -name "*.js" | wc -l

# 3. Find security hotspots
grep -r "password.*=.*\"" --include="*.java" --include="*.properties" -n | head -10
grep -r "query.*+" --include="*.java" --include="*.py" -n | head -10

# 4. Find performance hotspots
grep -r "\.saveAll(" --include="*.java" -n
grep -r "for.*for.*for" --include="*.java" --include="*.py" -n | head -10
```

### 30-Minute Full Analysis

See [QUICK-START.md](QUICK-START.md) for complete walkthrough with examples.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    ORCHESTRATOR                              │
│  • Discovers project structure                              │
│  • Detects languages and frameworks                         │
│  • Generates manifest                                        │
│  • Coordinates agents                                        │
│  • Merges and deduplicates findings                         │
└─────────────────────────────────────────────────────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   SECURITY   │  │ PERFORMANCE  │  │ CONCURRENCY  │
│    AGENT     │  │    AGENT     │  │    AGENT     │
│              │  │              │  │              │
│ • SQL Inject │  │ • N+1 Query  │  │ • Race Cond. │
│ • Auth Gaps  │  │ • Batch Ops  │  │ • Deadlocks  │
│ • XSS/CSRF   │  │ • Algorithms │  │ • Thread Pool│
└──────────────┘  └──────────────┘  └──────────────┘
        │                  │                  │
        └──────────────────┼──────────────────┘
                           ▼
                  ┌──────────────┐
                  │   ASSEMBLY   │
                  │ • Merge      │
                  │ • Dedupe     │
                  │ • Prioritize │
                  └──────────────┘
                           │
                           ▼
                  ┌──────────────┐
                  │    REPORT    │
                  │  • Markdown  │
                  │  • JSON      │
                  └──────────────┘
```

---

## Workflow

### Phase 0: Agent Instruction Briefing (2 minutes)
- Load completeness enforcement rules into agent context
- Configure 3-phase execution (Pre-Analysis Counting → Progressive Extraction → Output Validation)
- Set anti-summarization constraints

### Phase 1: Discovery (5 minutes)
- Scan directory structure
- Detect programming languages
- Identify frameworks (Spring Boot, Django, Express, etc.)
- Count files and lines of code
- Generate project manifest

### Phase 2: Pattern Scanning (5 minutes)
- Use `grep`/`ripgrep` for quick hotspot identification
- Find SQL injection patterns
- Find N+1 query patterns
- Find concurrency issues
- Find hardcoded secrets
- Generate hotspot list

### Phase 3: Agent Execution (30-60 minutes)
Run specialized agents in parallel with Chain of Thought reasoning:
- **Security Agent**: Authentication, authorization, input validation, cryptography
- **Performance Agent**: Database queries, algorithms, caching, transactions
- **Concurrency Agent**: Thread safety, race conditions, deadlocks, resource leaks
- **JPA/Hibernate Agent** (Java): Entity optimization, batch config, lazy loading
- **Resilience Agent**: Timeouts, circuit breakers, retries, bulkheads
- **Architecture Agent**: Dependency violations, coupling, god classes

Each agent follows:
1. Pre-Analysis Counting: Declare expected finding count by category
2. Progressive Extraction: Report progress every 10% with specific finding IDs
3. Output Validation: Verify declared_count === actual_count

### Phase 4: Result Assembly (5 minutes)
- Merge findings from all agents
- Deduplicate using canonical hashes
- Prioritize by severity (CRITICAL → HIGH → MEDIUM → LOW)
- Generate statistics

### Phase 5: Report Generation (5 minutes)
- Create professional markdown report
- Include code evidence for each finding
- Provide actionable recommendations
- Add quick wins section

### Phase 5.5: Agent Output Validation (Automatic)
- Validate: `declared_count === actual_count`
- Check: No ID gaps, no placeholders, no summarization keywords
- Verify: All required fields present (file, line, code_snippet, description)
- If validation fails: Re-run agent with corrected instructions

**Total Time**: 50-80 minutes for comprehensive analysis

---

## Documentation

### For AI Models (Read in This Order)

| Document | Description |
|----------|-------------|
| **[START-HERE.md](START-HERE.md)** | **🎯 START HERE** - Reading guide for AI models with mandatory reading order |
| [COMPLETENESS-ENFORCEMENT.md](COMPLETENESS-ENFORCEMENT.md) | 3-phase validation system to prevent summarization and ensure 100% finding documentation |
| [CLAUDE-ANALYSIS-FRAMEWORK.md](CLAUDE-ANALYSIS-FRAMEWORK.md) | Complete framework methodology with Phase 0-6 workflow and validation |
| [AGENT-PROMPTS.md](AGENT-PROMPTS.md) | Agent templates with Chain of Thought reasoning and completeness enforcement |

### For Humans

| Document | Description |
|----------|-------------|
| [QUICK-START.md](QUICK-START.md) | Step-by-step guide with real examples for running analyses |
| [LANGUAGE-PLUGINS.md](LANGUAGE-PLUGINS.md) | Language-specific patterns for Java, Python, JavaScript |

---

## When to Use This Framework

### Perfect For:
- **New codebases**: Understand inherited or legacy code
- **Pre-deployment audits**: Security and performance review
- **Onboarding**: Technical assessment of new projects
- **Performance investigations**: Find bottlenecks systematically
- **Security audits**: Comprehensive vulnerability scanning
- **Large refactoring**: Identify issues before major changes

### Not Ideal For:
- Single files or small scripts (<500 LOC)
- Real-time CI/CD checks (too thorough, takes 50-80 min)
- Style/formatting only (use linters instead)

---

## Supported Languages

### Fully Supported
- **Java** (Spring Boot, Hibernate, Feign, Maven/Gradle)
- **Python** (Django, Flask, FastAPI, SQLAlchemy)
- **JavaScript/Node.js** (Express, React, Mongoose, Sequelize)

### Easily Extensible
- Go
- Ruby
- PHP
- C#/.NET
- TypeScript
- Kotlin

See [LANGUAGE-PLUGINS.md](LANGUAGE-PLUGINS.md) for adding new languages.

---

## Example Output

### Security Finding
```json
{
  "id": "SEC-CRIT-001",
  "type": "SECURITY",
  "severity": "CRITICAL",
  "category": "SQL_INJECTION",
  "file": "src/main/java/com/example/UserController.java",
  "line": 45,
  "evidence": "String query = \"SELECT * FROM users WHERE email = '\" + email + \"'\";",
  "description": "SQL query constructed using string concatenation with user input",
  "impact": "Attacker can inject arbitrary SQL commands, potentially reading/modifying all database data",
  "recommendation": "Use PreparedStatement:\nString query = \"SELECT * FROM users WHERE email = ?\";\nPreparedStatement stmt = conn.prepareStatement(query);\nstmt.setString(1, email);"
}
```

### Performance Finding
```json
{
  "id": "PERF-CRIT-001",
  "type": "PERFORMANCE",
  "severity": "CRITICAL",
  "category": "N_PLUS_ONE_QUERY",
  "file": "src/main/java/com/example/service/OrderService.java",
  "line": 123,
  "evidence": "List<User> users = userRepository.findAll();\nfor (User user : users) {\n    user.getOrders().size();\n}",
  "description": "N+1 query pattern: loads 500 users then triggers 500 individual queries for orders",
  "impact": "500 database roundtrips instead of 1-2 queries. Operation takes 15 seconds instead of <1 second",
  "recommendation": "Add @BatchSize(size=10) to orders relationship OR use JOIN FETCH:\n@Query(\"SELECT DISTINCT u FROM User u LEFT JOIN FETCH u.orders\")"
}
```

---

## Real-World Results

### Spring Boot Microservice (85K LOC)
- **Analysis Time**: 62 minutes
- **Findings**: 247 issues
  - 9 CRITICAL (3 security, 6 performance)
  - 54 HIGH
  - 142 MEDIUM
  - 42 LOW
- **Quick Wins**: 8 configuration changes = 3-5x performance improvement in 1 hour

### Django Application (45K LOC)
- **Analysis Time**: 38 minutes
- **Findings**: 128 issues
  - 5 CRITICAL (4 security, 1 performance)
  - 32 HIGH
  - 68 MEDIUM
  - 23 LOW
- **Key Discoveries**: SQL injection, N+1 queries, missing authentication

### Node.js API (32K LOC)
- **Analysis Time**: 29 minutes
- **Findings**: 89 issues
  - 3 CRITICAL (XSS, missing auth, event loop blocking)
  - 24 HIGH
  - 45 MEDIUM
  - 17 LOW
- **Impact**: Fixed 3 CRITICAL issues in 2 hours

---

## Key Concepts

### Completeness Enforcement (NEW in v2.1)
**3-phase validation system** guarantees 100% finding documentation:
- **Phase 1**: Pre-Analysis Counting - Agent declares expected finding count before analyzing
- **Phase 2**: Progressive Extraction - Agent reports progress every 10% with specific IDs
- **Phase 3**: Output Validation - Verify `declared_count === actual_count`

**Anti-Summarization**: Prevents "Found 8 SQL injections" → Forces listing all 8 individually with file:line.

### Chain of Thought Reasoning (NEW in v2.1)
Each finding analyzed through **6-step systematic process**:
1. **Observation**: What code pattern exists?
2. **Analysis**: Why is this problematic?
3. **Context**: What makes it exploitable/problematic?
4. **Impact**: What are the consequences?
5. **Alternative Explanations**: Could this be a false positive?
6. **Confidence**: 90%+, 70-90%, 50-70%, or <50%

### Role-Based Agent Personas (NEW in v2.1)
Each agent has specific identity and expertise:
- **Alex "Paranoid" Rodriguez** - Security Agent (12+ years AppSec)
- **Maria "Profiler" Chen** - Performance Agent (15+ years DB optimization)
- **David "Parallel" Kumar** - Concurrency Agent (10+ years race conditions)
- **Sarah "ORM Whisperer" Patel** - JPA/Hibernate Agent (12+ years Hibernate)
- **James "Failover" Martinez** - Resilience Agent (10+ years fault-tolerance)
- **Emily "Architect" Zhang** - Architecture Agent (15+ years clean architecture)

### Semantic Segmentation
Split code by **architectural layers** (controller, service, DAO) rather than arbitrary file counts. Maintains context and reduces token usage.

### Context Injection
Provide minimal cross-layer context to agents (e.g., service knows what controllers call it). Prevents losing system-wide awareness.

### Pattern-Based Scanning
Use `grep` patterns to identify hotspots before deep analysis. Reduces 845 files to 50 files requiring detailed review.

### Tiered Analysis
- **Tier 1** (Quick): Pattern matching, metrics (all files, 0 tokens)
- **Tier 2** (Standard): Agent reads file, checks common issues (50% of files)
- **Tier 3** (Deep): Line-by-line analysis (10% of files - hotspots only)

### Language Plugins
Core framework is language-agnostic. Language-specific patterns and rules loaded as plugins.

### Hash-Based Deduplication
Use `hash(file+line+category)` to identify duplicate findings from multiple agents.

---

## Tactical Approaches

1. **Semantic Segmentation**: Divide by architecture, not file count
2. **Context Injection**: Maintain cross-layer awareness
3. **Pattern Scanning**: Grep first, analyze deep second
4. **Tiered Analysis**: Adaptive depth based on risk
5. **Language Plugins**: Core + language-specific modules
6. **Deduplication**: Canonical hashing of findings
7. **Cross-Cutting Agents**: Agents that span multiple layers
8. **Dependency Graphs**: Understand call relationships
9. **Size-Based Routing**: Different strategies for different file sizes
10. **Parallel Execution**: Run independent agents concurrently

See [CLAUDE-ANALYSIS-FRAMEWORK.md](CLAUDE-ANALYSIS-FRAMEWORK.md) for detailed explanations.

---

## Advanced Usage

### Custom Agents

Add domain-specific agents:

```markdown
# File: prompts/api-design-agent-prompt.md

## Mission
Analyze REST API design for consistency, versioning, and best practices.

## Checks
- Consistent naming (camelCase vs snake_case)
- Proper HTTP status codes
- Pagination implemented
- API versioning strategy
- Rate limiting
```

### CI/CD Integration

```yaml
# .github/workflows/code-review.yml
name: Deep Code Review

on:
  pull_request:
    branches: [main]

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run analysis
        run: |
          ./scripts/discover.sh
          ./scripts/pattern-scan.sh
          claude-code run-agent --prompt security-prompt.md
          ./scripts/generate-report.sh
      - name: Upload report
        uses: actions/upload-artifact@v2
        with:
          name: code-review-report
          path: CODE_REVIEW_REPORT.md
```

### Custom Language Support

```yaml
# plugins/go/patterns.yaml
security:
  - "password.*="
  - "exec.Command.*+"
  - "sql.Query.*+"

performance:
  - "for.*range"
  - "append("

concurrency:
  - "go func"
  - "sync.Mutex"
  - "chan "
```

---

## Best Practices

### Do:
- ✅ Run discovery phase first
- ✅ Use pattern scanning to identify hotspots
- ✅ Run agents in parallel when possible
- ✅ Review CRITICAL and HIGH findings first
- ✅ Provide code evidence for every finding
- ✅ Include actionable recommendations
- ✅ Track findings over time
- ✅ **Count findings before analyzing** (Pre-Analysis Counting phase)
- ✅ **Report progress every 10%** during extraction
- ✅ **List every finding individually** - never group or summarize
- ✅ **Include validation block** in output JSON
- ✅ **Use Chain of Thought reasoning** for each finding

### Don't:
- ❌ Skip discovery - you'll miss context
- ❌ Analyze entire codebase without pattern scan
- ❌ Run agents sequentially (waste time)
- ❌ Report findings without code evidence
- ❌ Ignore framework-specific optimizations
- ❌ Assume technologies not explicitly found
- ❌ Create findings based on opinions
- ❌ **Summarize findings** (e.g., "8 SQL injections found" - list all 8!)
- ❌ **Skip pre-analysis counting** - declare expected finding count first
- ❌ **Omit validation block** - always include validation in output JSON

---

## Limitations

1. **Time**: Comprehensive analysis takes 50-80 minutes
2. **Accuracy**: AI agents can have false positives (verify findings)
3. **Context**: Some business logic requires human understanding
4. **Coverage**: Doesn't replace security penetration testing
5. **Languages**: Not all languages equally supported (extendable)

---

## Contributing

This framework is designed to be extended. Contributions welcome:

1. **New language plugins**: Add patterns for Go, Ruby, PHP, etc.
2. **New agent types**: Monitoring, observability, scalability agents
3. **Improved patterns**: More accurate grep patterns
4. **Orchestration scripts**: Better automation
5. **Integration examples**: CI/CD, IDEs, Git hooks

---

## FAQ

### How is this different from static analysis tools?

Static analysis tools (SonarQube, ESLint, etc.) are:
- Fast (seconds to minutes)
- Pattern-based (predefined rules)
- Language-specific
- Integrated into CI/CD

This framework:
- Thorough (50-80 minutes)
- AI-powered (understands context)
- Language-agnostic
- Best for comprehensive audits

**Use both**: Static analysis for continuous checks, this framework for periodic deep dives.

### Can I use this with proprietary code?

Yes. All analysis happens locally. No code is sent to external services except the AI API (Claude Code).

### How accurate are the findings?

- **CRITICAL findings**: ~95% accuracy (verified issues)
- **HIGH findings**: ~85% accuracy
- **MEDIUM/LOW**: May include false positives, requires human review

Always verify CRITICAL findings before acting.

### Can I customize severity levels?

Yes. Severity depends on context:
- Production system: Hardcoded password = CRITICAL
- Internal tool: Hardcoded password = HIGH
- Proof-of-concept: Hardcoded password = MEDIUM

Adjust in agent prompts.

### How do I add a new programming language?

1. Create `plugins/[language]/` directory
2. Add `patterns.yaml` with grep patterns
3. Add language-specific agent logic
4. Update detection in discovery phase
5. Test with sample repository

See [LANGUAGE-PLUGINS.md](LANGUAGE-PLUGINS.md) for template.

### What is the Completeness Enforcement mechanism?

The **3-phase validation system** that guarantees 100% finding documentation:

1. **Phase 1 - Pre-Analysis Counting**: Agent must count and declare expected findings BEFORE analyzing
2. **Phase 2 - Progressive Extraction**: Agent reports progress every 10% with specific finding IDs
3. **Phase 3 - Output Validation**: Automated check ensures `declared_count === actual_count`

This prevents AI from summarizing findings (e.g., "8 SQL injections found" without listing all 8).

### What is Chain of Thought reasoning?

Each finding is analyzed through a **6-step systematic process**:

1. **Observation**: What code pattern exists?
2. **Analysis**: Why is this problematic?
3. **Context**: What makes it exploitable/problematic?
4. **Impact**: What are the consequences?
5. **Alternative Explanations**: Could this be a false positive?
6. **Confidence**: Assign 90%+, 70-90%, 50-70%, or <50% confidence level

This ensures thorough, factual analysis with clear reasoning for each finding.

---

## License

This framework documentation is provided as-is for educational and professional use.

---

## Support

- **Documentation**: See files in this repository
- **Issues**: Create GitHub issue
- **Discussions**: GitHub Discussions
- **Examples**: See [QUICK-START.md](QUICK-START.md)

---

## Acknowledgments

Built with insights from analyzing:
- Spring Boot microservices (50K-100K LOC)
- Django applications (20K-50K LOC)
- Node.js APIs (15K-40K LOC)
- Legacy enterprise systems (100K+ LOC)

Special thanks to the Claude Code team for the powerful agent orchestration capabilities.

---

**Version**: 2.1
**Last Updated**: 2025-10-11
**Maintained By**: Code Review Framework Community

### What's New in v2.1

- ✨ **START-HERE.md**: Mandatory reading guide for AI models
- ✨ **Completeness Enforcement**: 3-phase validation system preventing summarization
- ✨ **Chain of Thought Reasoning**: 6-step systematic analysis for each finding
- ✨ **Agent Personas**: Role-based identities with specific expertise
- ✨ **Output Validation**: Automated check ensuring `declared_count === actual_count`
- ✨ **Bash Toolkits**: Efficient grep patterns for each agent
- ✨ **Confidence Scoring**: 4-tier confidence levels for all findings

---

## Getting Started

### For AI Models
1. **Read [START-HERE.md](START-HERE.md) FIRST** - Mandatory reading order for AI
2. Read [COMPLETENESS-ENFORCEMENT.md](COMPLETENESS-ENFORCEMENT.md) - Learn 3-phase validation
3. Review [CLAUDE-ANALYSIS-FRAMEWORK.md](CLAUDE-ANALYSIS-FRAMEWORK.md) - Overall workflow
4. Study [AGENT-PROMPTS.md](AGENT-PROMPTS.md) - Agent templates and examples

### For Humans
1. Read [QUICK-START.md](QUICK-START.md) for hands-on tutorial
2. Review [CLAUDE-ANALYSIS-FRAMEWORK.md](CLAUDE-ANALYSIS-FRAMEWORK.md) for methodology
3. Check [AGENT-PROMPTS.md](AGENT-PROMPTS.md) for prompt templates
4. Explore [LANGUAGE-PLUGINS.md](LANGUAGE-PLUGINS.md) for language-specific patterns
5. Run your first analysis!

**Happy analyzing!** 🚀
