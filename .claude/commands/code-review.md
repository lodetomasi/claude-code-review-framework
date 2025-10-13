# /code-review - Comprehensive Code Review

Execute comprehensive code review using the Claude Code Review Framework v3.0.

## CONTEXT LOADING

**Load these framework files first** (critical - DO NOT skip):

1. `COMPLETENESS-ENFORCEMENT.md` - Validation rules
2. `SAMPLING-RULES.md` - Canonical sampling logic
3. `GLOSSARY.md` - Framework terminology
4. `UNIVERSAL-CONTEXT-MANAGEMENT.md` - IF codebase >100K LOC
5. `ORCHESTRATOR-TEMPLATE.md` - Execution workflow

**IF project has `CLAUDE.md`**: Load it for project-specific rules.

---

## EXECUTION PROTOCOL

You are the **Orchestrator AI** - you coordinate multiple specialized agents to perform a complete code review.

### Your Responsibilities

1. **Execute ORCHESTRATOR-TEMPLATE.md workflow** (Phase 0-6)
2. **Launch specialized agents** (Security, Performance, Concurrency, Architecture)
3. **Validate agent outputs** against completeness rules
4. **Assemble final report** with all findings

### Critical Rules (Non-Negotiable)

**Completeness Enforcement**:
- ALL findings MUST be listed individually (no summarization)
- Validate: `actual_count` within `[min_estimate, max_estimate]`
- NO phrases like "Found N issues" without listing all N

**Sampling Rules** (see SAMPLING-RULES.md):
- CRITICAL findings: ALL in detailed format (never sampled)
- HIGH findings: ALL in detailed format (never sampled)
- MEDIUM/LOW: Count-based sampling

**Context Management**:
- IF codebase >100K LOC: use Progressive Writing Strategy MANDATORY
- Monitor context usage - activate emergency protocols at >95%
- Use Write-Clear-Continue pattern for large analyses

---

## WORKFLOW EXECUTION

### PHASE 0: Initialization

```bash
# Set target codebase
CODEBASE_PATH="{{current_directory}}"
OUTPUT_DIR="$CODEBASE_PATH/analysis-output"
mkdir -p "$OUTPUT_DIR"
```

**Action**: Run discovery commands from ORCHESTRATOR-TEMPLATE.md Phase 1

---

### PHASE 1: Discovery

**Execute** (see ORCHESTRATOR-TEMPLATE.md Section "PHASE 1"):

1. Count LOC: `find . -name "*.java" -o -name "*.py" -o -name "*.js" | xargs wc -l`
2. Detect technologies (languages, frameworks)
3. Identify architecture layers
4. Generate `manifest.json`

**Output**: Manifest file with project metadata

---

### PHASE 2: Pattern Scanning

**Execute** (see ORCHESTRATOR-TEMPLATE.md Section "PHASE 2"):

1. Security patterns (SQL injection, hardcoded secrets, missing auth)
2. Performance patterns (N+1 queries, missing indexes)
3. Concurrency patterns (race conditions, deadlocks)

**Output**: Hotspot files (`hotspots_*.txt`)

---

### PHASE 3: Strategy Selection

**Decision Logic**:

```
IF LOC < 50K AND findings < 100:
    → Standard Output Strategy

ELIF LOC >= 500K:
    → Strategic Sampling + Progressive Writing

ELSE:
    → Progressive Writing Strategy
```

**Communicate strategy choice to user** before launching agents.

---

### PHASE 4: Agent Execution (PARALLEL)

**Launch these agents concurrently** using subagent pattern (Anthropic 2025):

#### Security Agent

**Mission**: Find all security vulnerabilities

**Context**:
- Load: `manifest.json`, `CLAUDE.md`, `hotspots_sql_injection.txt`, `hotspots_secrets.txt`
- Rules: COMPLETENESS-ENFORCEMENT.md, SAMPLING-RULES.md
- Template: AGENT-PROMPTS.md "Security Agent" section

**Output**: `analysis-output/security_findings.md`

**Instructions**:
```
1. Pre-Analysis Estimation: grep scan + confidence interval [min, max]
2. Progressive Extraction: analyze ALL files, report progress every 10%
3. Output Validation: verify actual within range, apply v3.0 sampling
4. Chain of Thought: <thinking> blocks for each finding
```

#### Performance Agent

**Mission**: Find all performance bottlenecks

**Context**:
- Load: `manifest.json`, `CLAUDE.md`, `hotspots_n_plus_one.txt`
- Rules: COMPLETENESS-ENFORCEMENT.md, SAMPLING-RULES.md
- Template: AGENT-PROMPTS.md "Performance Agent" section

**Output**: `analysis-output/performance_findings.md`

**Instructions**: Same as Security Agent (customized for performance)

#### Concurrency Agent

**Mission**: Find all concurrency issues

**Context**:
- Load: `manifest.json`, `CLAUDE.md`
- Rules: COMPLETENESS-ENFORCEMENT.md, SAMPLING-RULES.md
- Template: AGENT-PROMPTS.md "Concurrency Agent" section

**Output**: `analysis-output/concurrency_findings.md`

**Instructions**: Same as Security Agent (customized for concurrency)

#### Architecture Agent

**Mission**: Find all architectural violations

**Context**:
- Load: `manifest.json`, `CLAUDE.md`
- Rules: COMPLETENESS-ENFORCEMENT.md, SAMPLING-RULES.md
- Template: AGENT-PROMPTS.md "Architecture Agent" section

**Output**: `analysis-output/architecture_findings.md`

**Instructions**: Same as Security Agent (customized for architecture)

---

### PHASE 5: Validation

**For EACH agent output file**:

```python
# Validation checklist
validate_agent_output(agent_file):
    ✓ File not empty
    ✓ Quick Reference Table present
    ✓ All CRITICAL findings detailed
    ✓ All HIGH findings detailed
    ✓ Count-based sampling applied correctly
    ✓ No ID sequence gaps
    ✓ No summarization phrases
    ✓ actual_count within [min, max] OR variance documented
```

**IF validation fails**: Re-run agent with corrected instructions.

---

### PHASE 6: Report Assembly

1. **Merge findings** from all domain files
2. **Generate statistics** (CRITICAL/HIGH/MEDIUM/LOW counts)
3. **Create final report**: `CODE_REVIEW_REPORT_v3.0.md`
4. **Calculate metrics**: Issues per KLOC, coverage percentage

**Report structure** (see ORCHESTRATOR-TEMPLATE.md Section "PHASE 6"):
- Executive Summary
- Domain Findings (by agent)
- Statistics table
- Analysis metadata

---

## OUTPUT TO USER

After completing all phases, provide user with:

**Summary**:
```markdown
✅ Code Review Complete

**Project**: {{project_name}}
**LOC Analyzed**: {{total_loc}}
**Strategy**: {{strategy_used}}

**Findings**:
- CRITICAL: {{critical_count}} (ALL detailed)
- HIGH: {{high_count}} (ALL detailed)
- MEDIUM: {{medium_count}} ({{medium_detailed}} detailed + {{medium_table}} in Quick Ref)
- LOW: {{low_count}} ({{low_detailed}} detailed + {{low_table}} in Quick Ref)

**Total Documented**: {{total_findings}} (100% completeness ✓)

**Report Location**: `analysis-output/CODE_REVIEW_REPORT_v3.0.md`

**Next Steps**:
1. Review Section 3 (CRITICAL & HIGH issues) - fix immediately
2. Triage MEDIUM issues for next sprint
3. Backlog LOW issues
```

**Provide navigation guidance**:
- "Read Executive Summary first (2 min)"
- "Jump to Section 3 for actionable issues"
- "Use Quick Reference Tables to scan MEDIUM/LOW"

---

## ANTHROPIC 2025 BEST PRACTICES

**Research-Plan-Execute**:
- Agents read files FIRST, do NOT code immediately
- Plan approach before analysis
- Execute systematically

**Subagent Isolation**:
- Each agent has independent context window
- Pass only relevant information back to orchestrator
- Parallel execution for performance

**Chain of Thought**:
- MANDATORY <thinking> blocks for findings
- Observation → Hypothesis → Evidence → Impact → Confidence

**Permission Management**:
- Read-only operations by default
- Explicit user confirmation for writes
- No git operations without approval

---

## TROUBLESHOOTING

**Context Overflow**:
→ Switch to Progressive Writing immediately
→ Initialize domain files, write every N findings
→ Clear memory after each write

**Summarization Detected**:
→ Re-run agent with COMPLETENESS-ENFORCEMENT.md rules
→ Emphasize: "List EVERY finding individually"

**Count Mismatch**:
→ Check for documented variance_reason
→ If missing: re-run with stricter validation

---

## SUCCESS CRITERIA

✅ All phases 0-6 completed
✅ All agents returned valid output
✅ No completeness violations
✅ All CRITICAL/HIGH findings detailed
✅ Quick Reference Tables present where needed
✅ Final report generated
✅ Context usage <95% throughout
✅ User receives clear next steps

---

**Framework Version**: 3.0
**Slash Command**: `/code-review`
**Last Updated**: 2025-10-13

---

**BEGIN EXECUTION** when user invokes this command.
