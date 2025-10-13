# PROJECT INTEGRATION GUIDE - CLAUDE.md Integration

**Version**: 3.0 (AI-Optimized)
**Purpose**: How to integrate framework with project-specific CLAUDE.md files
**Last Updated**: 2025-10-13

---

## 🎯 OVERVIEW

This guide explains how the **Claude Code Review Framework** (generic, reusable) integrates with **CLAUDE.md** (project-specific rules).

**Key Concept**: Separation of Concerns

- **Framework rules** (this repo): HOW to analyze code (completeness, sampling, validation)
- **Project rules** (CLAUDE.md): WHAT to analyze (project conventions, architecture, business rules)

---

## 📁 FILE ROLES

### Framework Files (Generic - Applies to ALL Projects)

**Location**: `/path/to/claude-code-review-framework/`

**Purpose**: Define analysis methodology

**Key Files**:
- `COMPLETENESS-ENFORCEMENT.md` - 100% finding documentation rules
- `SAMPLING-RULES.md` - Count-based sampling logic
- `GLOSSARY.md` - Framework terminology
- `ORCHESTRATOR-TEMPLATE.md` - Execution workflow
- `AGENT-PROMPTS.md` - Agent templates

**Used By**: Orchestrator AI (you, when coordinating analysis)

**Never Modified**: These are stable, reusable across all projects

---

### CLAUDE.md (Project-Specific - One Per Project)

**Location**: `{{project_root}}/CLAUDE.md`

**Purpose**: Define project conventions and custom rules

**Auto-Loaded**: Yes - Claude Code automatically loads this file when starting conversation in project directory

**Example Content**:

```markdown
# Project: sport-gestione-licenze-service

## Build Commands
mvn clean install -DskipTests

## Test Commands
mvn test

## Architecture
- Layered microservice pattern
- Controllers: src/main/java/**/controller/
- Services: src/main/java/**/service/
- DAOs: src/main/java/**/dao/

## Project-Specific Rules
- All endpoints MUST have @PreAuthorize annotation
- All passwords MUST use BCrypt (minimum cost 12)
- No god classes > 500 LOC
- Drools rules files: src/main/resources/conf-drools/

## Code Review Focus Areas
- Hibernate N+1 queries (common issue in this codebase)
- Missing circuit breakers on Feign clients
- Unclosed JasperReports streams
```

**Used By**: All agents (Security, Performance, etc.)

**Modified**: Per-project customization

---

## 🔗 INTEGRATION PATTERN

### How Framework + CLAUDE.md Work Together

**Orchestrator loads BOTH contexts**:

```
┌─────────────────────────────────────────────────────────┐
│                    ORCHESTRATOR AI                       │
├─────────────────────────────────────────────────────────┤
│  Context Loaded:                                         │
│  1. Framework Rules (generic methodology)                │
│     - COMPLETENESS-ENFORCEMENT.md                        │
│     - SAMPLING-RULES.md                                  │
│     - GLOSSARY.md                                        │
│  2. Project Rules (specific conventions)                 │
│     - CLAUDE.md (auto-loaded by Claude Code)            │
│  3. Generated Context                                    │
│     - manifest.json (Phase 1 Discovery)                 │
│     - hotspots_*.txt (Phase 2 Pattern Scanning)         │
└─────────────────────────────────────────────────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   SECURITY   │  │ PERFORMANCE  │  │ CONCURRENCY  │
│    AGENT     │  │    AGENT     │  │    AGENT     │
│              │  │              │  │              │
│ Applies:     │  │ Applies:     │  │ Applies:     │
│ - Framework  │  │ - Framework  │  │ - Framework  │
│   rules      │  │   rules      │  │   rules      │
│ - Project    │  │ - Project    │  │ - Project    │
│   rules      │  │   rules      │  │   rules      │
└──────────────┘  └──────────────┘  └──────────────┘
```

---

## 📋 INTEGRATION WORKFLOW

### Step 1: Framework Setup (One Time)

```bash
# Clone framework repository
git clone https://github.com/your-org/claude-code-review-framework.git
cd claude-code-review-framework

# Verify files
ls -la
# Should see: START-HERE.md, COMPLETENESS-ENFORCEMENT.md, SAMPLING-RULES.md, etc.
```

### Step 2: Project Setup (Per Project)

```bash
# Navigate to your project
cd /path/to/your/project

# Create CLAUDE.md if not exists
touch CLAUDE.md

# Edit with project-specific rules
code CLAUDE.md
```

**CLAUDE.md Template**:

```markdown
# {{Project Name}}

## Build & Test
[Your build commands]

## Architecture
[Your architecture description]

## Project-Specific Code Review Rules
[Custom rules for this project]

## Known Issues
[Common problems to watch for]

## Exclusions
[Paths to skip during analysis]
```

### Step 3: Execute Code Review

**Option A: Using Slash Command** (Recommended)

```bash
# In project root
/code-review
```

Claude Code will:
1. Auto-load `CLAUDE.md` from current directory
2. Load framework rules from framework repo
3. Execute ORCHESTRATOR-TEMPLATE.md workflow
4. Apply both framework + project rules

**Option B: Manual Orchestration**

```markdown
# Orchestrator Prompt

You are the Orchestrator AI for code review.

**Load Framework Rules**:
1. /path/to/framework/COMPLETENESS-ENFORCEMENT.md
2. /path/to/framework/SAMPLING-RULES.md
3. /path/to/framework/GLOSSARY.md
4. /path/to/framework/ORCHESTRATOR-TEMPLATE.md

**Load Project Rules**:
- CLAUDE.md (current directory - auto-loaded)

**Execute Workflow**:
- Follow ORCHESTRATOR-TEMPLATE.md Phase 0-6
- Apply framework completeness rules
- Apply project-specific rules from CLAUDE.md

BEGIN
```

---

## 🎯 CONTEXT PRIORITY (When Rules Conflict)

### Hierarchy

```
1. CLAUDE.md (Project-Specific) - HIGHEST PRIORITY
   ↓
2. Framework Rules (Generic Methodology)
   ↓
3. Agent Defaults
```

### Example Conflicts & Resolution

#### Conflict 1: Severity Level

**Framework Rule**: Hardcoded password = CRITICAL
**CLAUDE.md**: "Hardcoded test password in test/ directory = LOW"

**Resolution**: CLAUDE.md wins (project context matters)

#### Conflict 2: Exclusions

**Framework Rule**: Analyze all files
**CLAUDE.md**: "Skip generated/ directory"

**Resolution**: CLAUDE.md wins (project-specific exclusion)

#### Conflict 3: Sampling Strategy

**Framework Rule**: MEDIUM findings - count-based sampling
**CLAUDE.md**: "For this project, ALL MEDIUM findings detailed (no sampling)"

**Resolution**: CLAUDE.md wins (project requirement)

---

## 📖 WRITING EFFECTIVE CLAUDE.md

### Best Practices

#### DO:

✅ **Be Specific**
```markdown
BAD:  "Check for security issues"
GOOD: "All @PostMapping endpoints MUST have @PreAuthorize annotation"
```

✅ **Include Examples**
```markdown
## Code Review Rule: Authentication
- Endpoint: POST /api/users
- MUST have: @PreAuthorize("hasRole('ADMIN')")
- Example: ConcertiniController.java:42 (correct implementation)
```

✅ **Document Known Issues**
```markdown
## Known Issues in This Project
- N+1 queries common in LicPermessoService (historical issue)
- Missing circuit breakers on all Feign clients
- JasperReports streams often not closed
```

✅ **Specify Exclusions**
```markdown
## Exclusions
- generated/: Auto-generated code (DTOs from OpenAPI)
- target/: Build artifacts
- test/resources/fixtures/: Test data files
```

✅ **Link to Architecture Docs**
```markdown
## Architecture
See [ARCHITECTURE.md](docs/ARCHITECTURE.md) for full details.

Key points for code review:
- Layered architecture (Controller → Service → DAO)
- No cross-layer dependencies
- All database access via Repositories only
```

#### DON'T:

❌ **Don't Duplicate Framework Rules**
```markdown
BAD:  "List every finding individually (completeness rule)"
GOOD: (omit - this is framework responsibility)
```

❌ **Don't Be Vague**
```markdown
BAD:  "Code should be clean"
GOOD: "Classes > 500 LOC flagged as MEDIUM (refactoring needed)"
```

❌ **Don't Include Temporary Notes**
```markdown
BAD:  "TODO: Update this section later"
GOOD: (remove incomplete sections)
```

---

## 🔍 VALIDATION CHECKLIST

### For Orchestrator (Before Launching Agents)

```markdown
Pre-Execution Validation:

□ Framework files loaded?
  - COMPLETENESS-ENFORCEMENT.md
  - SAMPLING-RULES.md
  - GLOSSARY.md
  - ORCHESTRATOR-TEMPLATE.md

□ CLAUDE.md loaded? (if exists in project)

□ Contexts merged correctly?
  - Framework rules understood
  - Project rules understood
  - No conflicting instructions unclear

□ Project-specific rules clear?
  - Custom severity levels documented
  - Exclusions list present
  - Architecture understood

□ Ready to launch agents?
```

### For Agents (During Analysis)

```markdown
Agent Context Validation:

□ Framework completeness rules active?
  - NO summarization
  - ALL findings listed
  - Validation block included

□ Project rules from CLAUDE.md active?
  - Custom severity adjustments applied
  - Project-specific patterns checked
  - Exclusions respected

□ Chain of Thought (Anthropic 2025)?
  - <thinking> blocks present
  - Reasoning documented

□ Output conforms to v3.0 Unified Strategy?
  - CRITICAL/HIGH: ALL detailed
  - MEDIUM/LOW: Count-based sampling
  - Quick Reference Table if needed
```

---

## 🛠️ TROUBLESHOOTING

### Issue: CLAUDE.md Not Loaded

**Symptoms**:
- Agent output doesn't follow project-specific rules
- Generic patterns used instead of project conventions

**Solution**:
```bash
# Verify CLAUDE.md exists
ls -la CLAUDE.md

# Verify in project root (not subdirectory)
pwd
# Should output: /path/to/your/project

# Reload conversation (Claude Code will auto-load on new conversation)
```

### Issue: Framework Rules Not Applied

**Symptoms**:
- Summarization detected in output
- Count validation missing

**Solution**:
```markdown
# Explicit orchestrator instruction

CRITICAL: Before launching agents, load framework rules:
1. Read COMPLETENESS-ENFORCEMENT.md
2. Read SAMPLING-RULES.md
3. Read GLOSSARY.md

Then proceed with analysis.
```

### Issue: Conflicting Rules Unclear

**Symptoms**:
- Agent asks "Which rule to follow?"
- Inconsistent severity assignments

**Solution**:
```markdown
Update CLAUDE.md with explicit priority:

## Rule Priority
When framework and project rules conflict:
- Project rules (this file) take precedence
- Example: Framework says CRITICAL, project says HIGH → use HIGH
```

---

## 📊 INTEGRATION EXAMPLES

### Example 1: Spring Boot Microservice

**Project**: sport-gestione-licenze-service
**LOC**: 138K
**Strategy**: Progressive Writing

**CLAUDE.md Excerpt**:
```markdown
# sport-gestione-licenze-service

## Architecture
- Spring Boot 3.5.5 microservice
- Drools 10.1.0 rules engine
- Oracle database via JPA/Hibernate

## Code Review Rules
- ALL Drools .drl files: check for rule conflicts
- Feign clients: MUST have circuit breaker (@EnableCircuitBreaker)
- Hibernate queries: check for N+1 patterns (common in this codebase)
- Missing @PreAuthorize: flag as CRITICAL (not HIGH)

## Exclusions
- target/generated-sources/: QueryDSL generated code
- src/test/resources/: Test fixtures
```

**Integration Result**:
- Framework ensures 100% completeness
- Project rules customize severity (missing @PreAuthorize = CRITICAL)
- Exclusions prevent false positives on generated code

### Example 2: Python Django App

**Project**: django-ecommerce-api
**LOC**: 45K
**Strategy**: Standard Output

**CLAUDE.md Excerpt**:
```markdown
# django-ecommerce-api

## Architecture
- Django 4.2 REST API
- PostgreSQL database
- Celery for async tasks

## Code Review Rules
- All API views: check for permission_classes
- Raw SQL queries: flag as CRITICAL (ORM preferred)
- Celery tasks: check for timeout configuration
- Missing rate limiting: flag as HIGH

## Known Issues
- Legacy code in payments/ has technical debt
- Migrations/ directory has merge conflicts (skip)
```

**Integration Result**:
- Framework handles small codebase efficiently
- Project rules catch Django-specific issues
- Known issues documented → informed analysis

---

## 🎯 SUCCESS CRITERIA

**Integration is successful when**:

✅ Orchestrator loads both framework + project context
✅ Agents apply framework completeness rules
✅ Agents apply project-specific conventions
✅ Conflicts resolved according to hierarchy (CLAUDE.md wins)
✅ Exclusions respected
✅ Output conforms to v3.0 Unified Strategy
✅ Project-specific findings detailed correctly

---

## 📚 RELATED DOCUMENTATION

- **[START-HERE.md](START-HERE.md)**: Framework entry point
- **[ORCHESTRATOR-TEMPLATE.md](ORCHESTRATOR-TEMPLATE.md)**: Execution workflow
- **[GLOSSARY.md](GLOSSARY.md)**: Framework terminology
- **[.claude/commands/code-review.md](.claude/commands/code-review.md)**: Slash command for automated integration

---

**Version**: 3.0 (AI-Optimized)
**Last Updated**: 2025-10-13

---

END OF PROJECT INTEGRATION GUIDE
