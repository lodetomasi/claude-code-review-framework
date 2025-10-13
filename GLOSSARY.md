# FRAMEWORK GLOSSARY - CANONICAL TERMINOLOGY

**Version**: 3.0 (AI-Optimized)
**Status**: Single Source of Truth for Framework Terminology
**Last Updated**: 2025-10-13

---

## ⚠️ CRITICAL - CANONICAL REFERENCE

**This document defines the ONLY authorized terminology for the framework.**

All framework documents MUST use these exact terms, not variations or synonyms.

---

## 🎯 CORE CONCEPTS

### Analysis & Execution

**Agent**
: An autonomous AI entity executing a specialized code review task (Security Agent, Performance Agent, etc.)

**Orchestrator**
: The primary AI entity coordinating multiple agents, managing workflow phases, and assembling results

**Subagent**
: A specialized agent launched by another agent for parallel task execution (Anthropic 2025 pattern)

**Phase**
: A discrete step in the analysis workflow (Phase 0: Briefing, Phase 1: Discovery, etc.)

**Hotspot**
: A file or code section identified by pattern scanning as requiring deep analysis

**Finding**
: A single discovered issue with severity, file:line, evidence, and recommendation

**Manifest**
: Project metadata JSON generated in Phase 1 (languages, frameworks, architecture)

---

## 📊 OUTPUT & DOCUMENTATION

### Documentation Formats

**Detailed Format**
: 5-line finding format with File, Severity, Problem, Impact, Fix (used for CRITICAL/HIGH + samples)

**Quick Reference Table**
: Compact markdown table indexing findings (ID | Severity | Category | File:Line | Brief Description)

**Progressive Writing**
: Writing findings to disk during analysis (not at end) to prevent context overflow

**Write-Clear-Continue Pattern**
: Progressive Writing implementation: write batch → flush to disk → clear from memory → continue

**Incremental Writing**
: Synonym for Progressive Writing (DO NOT USE - use "Progressive Writing" instead)

### Sampling & Coverage

**Count-Based Sampling**
: v3.0 output strategy using finding counts to determine detail level (see SAMPLING-RULES.md)

**Strategic Sampling**
: Analyzing subset of codebase (40%+ for >500K LOC) prioritizing entry points and security paths

**Completeness**
: 100% finding discovery - ALL issues identified (even if not all detailed in output)

**Completeness Violation**
: When documented count ≠ found count (FATAL error)

**Summarization**
: Forbidden practice of grouping findings ("8 SQL injections found" without listing all 8)

---

## 🧠 CONTEXT MANAGEMENT

### Memory & Resources

**Context Window**
: Total token capacity available to AI model (200K tokens for Claude Sonnet 3.5)

**Context Usage**
: Current percentage of context window occupied

**Context Overflow**
: Exceeding context window limit (>95% usage triggers warnings)

**Context Overflow Threshold**
: 95% context usage (MANDATORY threshold for activating emergency protocols)

**Dynamic Write Interval**
: Adaptive batch size based on context usage (50/25/10/1 findings per batch)

**Write Frequency**
: Synonym for Dynamic Write Interval (DO NOT USE - use "Dynamic Write Interval" instead)

**Batch Threshold**
: Synonym for Dynamic Write Interval (DO NOT USE - use "Dynamic Write Interval" instead)

### Compression Techniques

**Progressive Compression**
: In-memory technique during analysis: first finding=full detail, rest=location only

**Compressed Format**
: Temporary in-memory representation (DO NOT confuse with Quick Reference Table)

**Semantic Segmentation**
: Dividing codebase by architectural layers (controller, service, DAO) not file counts

**Tiered Analysis**
: Adaptive depth (Tier 1: pattern scan, Tier 2: standard analysis, Tier 3: deep dive)

---

## 🎯 VALIDATION & QUALITY

### Estimation & Validation

**Pre-Analysis Estimation**
: Phase 1 requirement: declare confidence interval [min, max] before analyzing

**Pre-Analysis Counting**
: Deprecated v2.4 term (DO NOT USE - use "Pre-Analysis Estimation" instead)

**Confidence Interval**
: [min, max] range estimate for expected findings (e.g., [36, 77])

**Confidence Level**
: Uncertainty margin for estimation (e.g., ±35%)

**Actual Count**
: Total findings discovered after analysis completes

**Variance**
: Difference between actual count and estimated midpoint

**Variance Reason**
: Required documentation when actual count falls outside estimated range

### Validation Rules

**Three-Phase Validation**
: v3.0 completeness system: 1) Pre-Analysis Estimation → 2) Progressive Extraction → 3) Output Validation

**Chain of Thought**
: Anthropic 2025 requirement: explicit reasoning steps for each finding (Observation → Hypothesis → Evidence → Impact → Confidence)

**Validation Block**
: JSON section in agent output verifying completeness and sampling correctness

**ID Sequence Gap**
: Missing finding ID in sequence (e.g., SEC-001, SEC-003 missing SEC-002) - FATAL error

---

## 🔧 STRATEGIES & PATTERNS

### Analysis Strategies

**Standard Output Strategy**
: For <100K LOC codebases: agents return complete findings in JSON response

**Progressive Writing Strategy**
: For >100K LOC codebases: agents write to disk during analysis, return summary only

**Hybrid Strategy**
: Deprecated v2.3 term (DO NOT USE - use either "Standard" or "Progressive Writing")

**v3.0 Unified Strategy**
: Count-based sampling rules (CRITICAL/HIGH=ALL, MEDIUM/LOW=conditional) - see SAMPLING-RULES.md

**v2.4 Output Strategy**
: Deprecated predecessor of v3.0 Unified Strategy

### Workflow Patterns

**Research-Plan-Execute**
: Anthropic 2025 workflow: agents read files first, plan approach, then code

**Parallel Execution**
: Running multiple agents concurrently (Security + Performance + Concurrency simultaneously)

**Sequential Execution**
: Running agents one after another (less efficient, avoid when possible)

**Agent Isolation**
: Each agent has independent context window (Anthropic 2025 pattern)

---

## 📁 FILES & DIRECTORIES

### Framework Files

**START-HERE.md**
: Mandatory entry point for AI execution (reading order, strategy selection)

**COMPLETENESS-ENFORCEMENT.md**
: Three-phase validation system specification

**UNIVERSAL-CONTEXT-MANAGEMENT.md**
: Memory optimization strategies for large codebases

**AGENT-PROMPTS.md**
: Agent template library with Chain of Thought requirements

**FRAMEWORK-RULES-HIERARCHY.md**
: Conflict resolution priority order (Completeness > Context Management > Output Strategy)

**SAMPLING-RULES.md**
: Canonical sampling logic (count-based rules, dynamic intervals)

**GLOSSARY.md**
: This document - canonical terminology reference

**ORCHESTRATOR-TEMPLATE.md**
: Practical execution guide for orchestrating agents

**LANGUAGE-PLUGINS.md**
: Language-specific patterns (Java, Python, JavaScript, etc.)

**EXAMPLES.md**
: Real-world code review examples

**SCRIPTS.md**
: Ready-to-use bash scripts for analysis tasks

### Project Files

**CLAUDE.md**
: Project-specific instructions file auto-loaded by Claude Code

**manifest.json**
: Project metadata generated in Phase 1 Discovery

**.claude/commands/**
: Directory for custom slash commands (markdown templates)

**{domain}_findings.md**
: Agent output files (security_findings.md, performance_findings.md, etc.)

**CODE_REVIEW_REPORT_v3.0.md**
: Final consolidated report with all findings

---

## 🚨 SEVERITY LEVELS

**CRITICAL**
: Data breach, system crash, security exploit - fix immediately (same day)

**HIGH**
: Significant performance degradation, authentication weakness - fix this sprint (1-2 weeks)

**MEDIUM**
: Code quality issue, minor performance concern - fix next sprint (2-4 weeks)

**LOW**
: Style improvement, minor optimization - backlog (when convenient)

---

## 📊 METRICS & THRESHOLDS

### Size Thresholds

**Small Codebase**
: <10K LOC - use Standard Output Strategy

**Medium Codebase**
: 10-50K LOC - use Standard Output Strategy

**Large Codebase**
: 50-100K LOC - use Progressive Writing Strategy recommended

**Very Large Codebase**
: 100-500K LOC - use Progressive Writing Strategy MANDATORY

**Extremely Large Codebase**
: >500K LOC - use Strategic Sampling + Progressive Writing

### Finding Count Thresholds

**Low Finding Count**
: <100 total findings - fits in Standard Output

**Medium Finding Count**
: 100-200 findings - Progressive Writing recommended

**High Finding Count**
: >200 findings - Progressive Writing MANDATORY

### Context Thresholds

**0-60% Context Usage**
: Normal - full details, no compression needed

**60-70% Context Usage**
: Moderate - start batching similar findings

**70-80% Context Usage**
: High - CRITICAL/HIGH only with minimal evidence

**80-90% Context Usage**
: Critical - CRITICAL findings only, use pattern codes

**90-95% Context Usage**
: Emergency - stop new analysis, prepare immediate output

**>95% Context Usage**
: Context Overflow - FATAL, must switch to Progressive Writing immediately

---

## 🔗 CROSS-REFERENCES

### When Terms Conflict

**If you see variations**, use this authoritative term:

| Deprecated / Variation | Canonical Term (USE THIS) |
|------------------------|---------------------------|
| Incremental Writing | Progressive Writing |
| Write-Clear Pattern | Write-Clear-Continue Pattern |
| Batch Threshold | Dynamic Write Interval |
| Write Frequency | Dynamic Write Interval |
| Pre-Analysis Counting | Pre-Analysis Estimation |
| Compressed Format (in output) | Quick Reference Table |
| Table-Only Findings | Quick Reference Table |
| Context Overflow (as threshold) | Context Overflow Threshold (95%) |
| Hybrid Strategy | Progressive Writing Strategy |
| v2.4 Output Strategy | v3.0 Unified Strategy |

### Adding New Terms

**To add terminology**:

1. Verify term doesn't already exist (check this glossary)
2. Define term clearly with concrete examples
3. Add to relevant section above
4. Update framework documents to use new term
5. Deprecate old terms if replacing

---

## 📖 USAGE GUIDELINES

### For AI Agents

**When analyzing code**:

- Use ONLY terms from this glossary
- If uncertain about term, reference this document
- Never invent new terminology without updating this file
- Use canonical terms in all output (findings, reports, logs)

### For Framework Authors

**When writing documentation**:

- Reference this glossary for correct terminology
- Link to GLOSSARY.md instead of defining terms inline
- Update glossary before introducing new concepts
- Mark deprecated terms clearly

### For Human Users

**When reading reports**:

- Consult this glossary for unfamiliar terms
- Expect consistent terminology across all documents
- Report terminology inconsistencies as framework bugs

---

## 🎯 VERIFICATION CHECKLIST

Use this to verify terminology compliance:

**During Document Writing**:

- [ ] All specialized terms defined in this glossary?
- [ ] No deprecated terms used?
- [ ] Canonical terms used consistently?
- [ ] Cross-references to glossary added?

**During Code Review**:

- [ ] Agent output uses canonical terminology?
- [ ] Report sections use consistent terms?
- [ ] No invented terms without glossary entry?

---

## 📊 TERMINOLOGY STATISTICS

| Category | Terms Defined |
|----------|---------------|
| Analysis & Execution | 7 |
| Output & Documentation | 8 |
| Context Management | 9 |
| Validation & Quality | 10 |
| Strategies & Patterns | 9 |
| Files & Directories | 12 |
| Severity Levels | 4 |
| Metrics & Thresholds | 14 |
| **TOTAL** | **73** |

---

## 🔄 VERSION HISTORY

**v3.0 (2025-10-13)**:

- Initial canonical glossary
- Standardized 73 framework terms
- Added Anthropic 2025 patterns
- Consolidated variations into canonical forms
- Deprecated v2.4 terminology

---

**Version**: 3.0 (AI-Optimized)
**Last Updated**: 2025-10-13
**Status**: Canonical Reference

---

END OF GLOSSARY
