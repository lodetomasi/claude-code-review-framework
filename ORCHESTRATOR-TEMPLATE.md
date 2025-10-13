# ORCHESTRATOR TEMPLATE - PRACTICAL EXECUTION GUIDE

**Version**: 3.0 (AI-Optimized)
**Purpose**: Step-by-step guide for executing code review framework
**Last Updated**: 2025-10-13

---

## ⚠️ BEFORE YOU START

**Mandatory Reading** (in this exact order):

1. **[START-HERE.md](START-HERE.md)** - Determine your strategy
2. **[COMPLETENESS-ENFORCEMENT.md](COMPLETENESS-ENFORCEMENT.md)** - Validation rules
3. **[SAMPLING-RULES.md](SAMPLING-RULES.md)** - Canonical sampling logic
4. **[GLOSSARY.md](GLOSSARY.md)** - Framework terminology

**IF** codebase >100K LOC **OR** expected findings >200:
- 5. **[UNIVERSAL-CONTEXT-MANAGEMENT.md](UNIVERSAL-CONTEXT-MANAGEMENT.md)** - Memory optimization

---

## 🎯 EXECUTION WORKFLOW

This is your **executable checklist** for running a complete code review.

---

### ✅ PRE-EXECUTION CHECKLIST

**Before launching any agents**:

```markdown
□ Framework documents read (see above)?
□ Target codebase path identified?
□ CLAUDE.md exists in target project (for project-specific rules)?
□ Output directory created (e.g., analysis-output/)?
□ Sufficient disk space (est. 100MB for 100K LOC analysis)?
```

---

## PHASE 0: INITIALIZATION (2 minutes)

### Step 0.1: Set Working Directory

```bash
# Set your target codebase path
export CODEBASE_PATH="/path/to/your/project"
export OUTPUT_DIR="$CODEBASE_PATH/analysis-output"

# Create output directory
mkdir -p "$OUTPUT_DIR"
cd "$CODEBASE_PATH"
```

### Step 0.2: Load Project Context

```bash
# Check if project has CLAUDE.md
if [ -f "CLAUDE.md" ]; then
    echo "✓ CLAUDE.md found - project-specific rules will be loaded"
    export PROJECT_RULES_FILE="CLAUDE.md"
else
    echo "⚠ No CLAUDE.md found - using framework defaults only"
fi
```

### Step 0.3: Validate Framework Readiness

```bash
# Verify framework files accessible
FRAMEWORK_DIR="/path/to/claude-code-review-framework"

for file in START-HERE.md COMPLETENESS-ENFORCEMENT.md SAMPLING-RULES.md GLOSSARY.md AGENT-PROMPTS.md; do
    if [ ! -f "$FRAMEWORK_DIR/$file" ]; then
        echo "✗ Missing: $file"
        exit 1
    fi
done

echo "✓ Framework files verified"
```

---

## PHASE 1: DISCOVERY (5-10 minutes)

### Step 1.1: Count Lines of Code

```bash
# Count total LOC by language
echo "Counting lines of code..."

find . -name "*.java" -o -name "*.py" -o -name "*.js" -o -name "*.go" -o -name "*.rb" | \
    xargs wc -l | tail -1 | awk '{print "Total LOC: " $1}'

# Store for strategy selection
export TOTAL_LOC=$(find . -name "*.java" -o -name "*.py" -o -name "*.js" | xargs wc -l | tail -1 | awk '{print $1}')

echo "TOTAL_LOC=$TOTAL_LOC"
```

### Step 1.2: Detect Technologies

```bash
# Detect languages
echo "Detecting technologies..."

LANGUAGES=()
[ -n "$(find . -name '*.java' -print -quit)" ] && LANGUAGES+=("Java")
[ -n "$(find . -name '*.py' -print -quit)" ] && LANGUAGES+=("Python")
[ -n "$(find . -name '*.js' -print -quit)" ] && LANGUAGES+=("JavaScript")
[ -n "$(find . -name '*.go' -print -quit)" ] && LANGUAGES+=("Go")

echo "Languages detected: ${LANGUAGES[@]}"

# Detect frameworks
FRAMEWORKS=()
grep -r "springframework" --include="*.xml" --include="*.java" -l > /dev/null 2>&1 && FRAMEWORKS+=("Spring Boot")
grep -r "from flask import" --include="*.py" -l > /dev/null 2>&1 && FRAMEWORKS+=("Flask")
grep -r "express()" --include="*.js" -l > /dev/null 2>&1 && FRAMEWORKS+=("Express")

echo "Frameworks detected: ${FRAMEWORKS[@]}"
```

### Step 1.3: Identify Architecture Layers

```bash
# Find architectural structure
echo "Analyzing architecture..."

LAYERS=()
[ -d "$(find . -type d -name 'controller*' -o -name 'controllers' -print -quit)" ] && LAYERS+=("controllers")
[ -d "$(find . -type d -name 'service*' -o -name 'services' -print -quit)" ] && LAYERS+=("services")
[ -d "$(find . -type d -name 'repository' -o -name 'repositories' -o -name 'dao' -print -quit)" ] && LAYERS+=("data-layer")
[ -d "$(find . -type d -name 'integration' -print -quit)" ] && LAYERS+=("integration")

echo "Architecture layers: ${LAYERS[@]}"
```

### Step 1.4: Generate Manifest

```bash
# Create manifest.json
cat > "$OUTPUT_DIR/manifest.json" <<EOF
{
  "project_name": "$(basename $CODEBASE_PATH)",
  "analysis_date": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "framework_version": "3.0",
  "languages": $(printf '%s\n' "${LANGUAGES[@]}" | jq -R . | jq -s .),
  "frameworks": $(printf '%s\n' "${FRAMEWORKS[@]}" | jq -R . | jq -s .),
  "total_loc": $TOTAL_LOC,
  "architecture": {
    "layers": $(printf '%s\n' "${LAYERS[@]}" | jq -R . | jq -s .)
  }
}
EOF

echo "✓ Manifest generated: $OUTPUT_DIR/manifest.json"
cat "$OUTPUT_DIR/manifest.json"
```

---

## PHASE 2: PATTERN SCANNING (5 minutes)

### Step 2.1: Security Patterns

```bash
echo "Scanning for security hotspots..."

# SQL injection patterns
grep -rn "String.*=.*SELECT\|executeQuery.*+" --include="*.java" --include="*.py" --include="*.js" \
    > "$OUTPUT_DIR/hotspots_sql_injection.txt" 2>/dev/null || echo "No SQL injection patterns found"

# Hardcoded secrets
grep -rn "password.*=.*\".*\"\|api_key.*=.*\"" --include="*.java" --include="*.py" --include="*.js" --include="*.yml" \
    > "$OUTPUT_DIR/hotspots_secrets.txt" 2>/dev/null || echo "No hardcoded secrets found"

# Missing authentication
grep -rn "@PostMapping\|@PutMapping\|@DeleteMapping" --include="*Controller.java" | \
    grep -v "@PreAuthorize" > "$OUTPUT_DIR/hotspots_missing_auth.txt" 2>/dev/null || echo "All endpoints have auth"

echo "✓ Security scan complete"
```

### Step 2.2: Performance Patterns

```bash
echo "Scanning for performance hotspots..."

# N+1 query patterns
grep -rn "for.*:.*\.\(get\|find\)\|forEach.*\->" --include="*Service.java" --include="*.py" \
    > "$OUTPUT_DIR/hotspots_n_plus_one.txt" 2>/dev/null || echo "No obvious N+1 patterns"

# Missing indexes (TODO queries)
grep -rn "TODO.*index\|FIXME.*performance" --include="*.java" --include="*.py" \
    > "$OUTPUT_DIR/hotspots_performance_todos.txt" 2>/dev/null || echo "No performance TODOs"

echo "✓ Performance scan complete"
```

### Step 2.3: Consolidate Hotspots

```bash
# Count hotspots
SECURITY_HOTSPOTS=$(cat "$OUTPUT_DIR/hotspots_"*.txt 2>/dev/null | wc -l)
echo "Total hotspots identified: $SECURITY_HOTSPOTS"

# Store for agent priority
export HOTSPOT_COUNT=$SECURITY_HOTSPOTS
```

---

## PHASE 3: STRATEGY SELECTION (1 minute)

### Step 3.1: Determine Strategy

```bash
# Strategy selection logic
if [ "$TOTAL_LOC" -lt 50000 ] && [ "$HOTSPOT_COUNT" -lt 100 ]; then
    export ANALYSIS_STRATEGY="Standard"
    echo "✓ Selected: Standard Output Strategy"
elif [ "$TOTAL_LOC" -ge 500000 ]; then
    export ANALYSIS_STRATEGY="Strategic Sampling"
    echo "✓ Selected: Strategic Sampling + Progressive Writing"
else
    export ANALYSIS_STRATEGY="Progressive Writing"
    echo "✓ Selected: Progressive Writing Strategy"
fi
```

---

## PHASE 4: AGENT EXECUTION (30-60 minutes)

### Step 4.1: Initialize Agent Output Files

**IF** using Progressive Writing Strategy:

```bash
# Create output files for each agent
for domain in security performance concurrency jpa resilience architecture; do
    cat > "$OUTPUT_DIR/${domain}_findings.md" <<EOF
# ${domain^} Findings

## Quick Reference Table

| ID | Severity | Category | File:Line | Brief Description |
|----|----------|----------|-----------|-------------------|

---

## Detailed Findings

EOF
    echo "✓ Initialized ${domain}_findings.md"
done
```

### Step 4.2: Prepare Agent Context

```bash
# Create agent briefing document
cat > "$OUTPUT_DIR/agent_context.md" <<EOF
# Agent Context

## Project Manifest
$(cat $OUTPUT_DIR/manifest.json)

## Analysis Strategy
$ANALYSIS_STRATEGY

## Hotspots Identified
$HOTSPOT_COUNT files requiring deep analysis

## Framework Rules
- Completeness Enforcement: MANDATORY
- Sampling Rules: See SAMPLING-RULES.md
- Output Strategy: v3.0 Unified

## Project-Specific Rules
$([ -f "$PROJECT_RULES_FILE" ] && cat "$PROJECT_RULES_FILE" || echo "None (using framework defaults)")

EOF

echo "✓ Agent context prepared"
```

### Step 4.3: Launch Agents (Parallel Execution)

**For Claude Code users** - use `/code-review` slash command (see `.claude/commands/code-review.md`)

**Manual execution** (example for Security Agent):

```markdown
**Prompt Template** (copy and customize):

---

# Security Agent - Code Review Task

## Your Role
You are Alex "Paranoid" Rodriguez, Senior Security Engineer with 12+ years in AppSec.

## Mission
Perform comprehensive security analysis of the codebase using framework rules.

## Context
**Load these files into your context**:
1. Project manifest: `analysis-output/manifest.json`
2. Project rules: `CLAUDE.md` (if exists)
3. Agent context: `analysis-output/agent_context.md`
4. Security hotspots: `analysis-output/hotspots_*.txt`

**Framework Rules**:
- Completeness Enforcement: [COMPLETENESS-ENFORCEMENT.md](COMPLETENESS-ENFORCEMENT.md)
- Sampling Rules: [SAMPLING-RULES.md](SAMPLING-RULES.md)
- Agent Template: [AGENT-PROMPTS.md](AGENT-PROMPTS.md) Section "Security Agent"

## Analysis Strategy
**$ANALYSIS_STRATEGY**

## Output File
Write findings to: `analysis-output/security_findings.md`

## Execution Instructions

### Phase 1: Pre-Analysis Estimation
1. Run preliminary grep scan for security patterns
2. Estimate finding count range: [min, max] with confidence level
3. Document estimation in output

### Phase 2: Progressive Extraction
1. Analyze files systematically
2. Report progress every 10% with finding IDs
3. IF Progressive Writing: write to disk every N findings (dynamic interval)
4. ALWAYS list every finding individually - NO summarization

### Phase 3: Output Validation
1. Verify actual_count within [min_estimate, max_estimate]
2. Apply v3.0 Unified Sampling (CRITICAL/HIGH=ALL, MEDIUM/LOW=count-based)
3. Validate: no ID gaps, all findings documented

## Chain of Thought Requirements (Anthropic 2025)
For EACH finding, include <thinking> blocks:
- Observation → Hypothesis → Evidence → Impact → Confidence

## Begin Analysis
[Agent starts executing here]

---
```

**Launch all agents in parallel** (if framework supports):
- Security Agent
- Performance Agent
- Concurrency Agent
- JPA/Hibernate Agent (if Java project)
- Resilience Agent
- Architecture Agent

### Step 4.4: Monitor Agent Progress

```bash
# Poll agent output files for progress
while true; do
    for file in $OUTPUT_DIR/*_findings.md; do
        FINDINGS_COUNT=$(grep -c "^### " "$file" 2>/dev/null || echo "0")
        echo "$(basename $file): $FINDINGS_COUNT findings"
    done
    sleep 30
done
```

---

## PHASE 5: VALIDATION (5 minutes)

### Step 5.1: Validate Agent Outputs

```bash
echo "Validating agent outputs..."

for file in $OUTPUT_DIR/*_findings.md; do
    echo "Validating $(basename $file)..."

    # Check: File not empty
    [ -s "$file" ] || echo "⚠ WARNING: Empty file"

    # Check: Has Quick Reference Table
    grep -q "Quick Reference Table" "$file" || echo "⚠ WARNING: Missing Quick Reference Table"

    # Check: Has detailed findings
    DETAILED_COUNT=$(grep -c "^### " "$file")
    echo "  Detailed findings: $DETAILED_COUNT"
done

echo "✓ Validation complete"
```

### Step 5.2: Verify Completeness

```bash
# Check for completeness violations
echo "Checking completeness..."

for file in $OUTPUT_DIR/*_findings.md; do
    # Look for forbidden summarization phrases
    if grep -qi "found.*issues\|multiple.*vulnerabilities\|similar problems in.*files" "$file"; then
        echo "✗ FATAL: Summarization detected in $(basename $file)"
        echo "  Re-run agent with completeness enforcement"
    fi
done

echo "✓ No completeness violations"
```

---

## PHASE 6: REPORT ASSEMBLY (5 minutes)

### Step 6.1: Merge Findings

```bash
echo "Assembling final report..."

cat > "$OUTPUT_DIR/CODE_REVIEW_REPORT_v3.0.md" <<EOF
================================================================================
                           CODE REVIEW REPORT
                     $(basename $CODEBASE_PATH)
================================================================================
Report ID: $(date +%Y-%m-%d)-$(uuidgen | cut -d'-' -f1)
Analysis Date: $(date +%Y-%m-%d)
Framework Version: 3.0
Type: Performance & Security Code Review

---

## Executive Summary

**Project**: $(basename $CODEBASE_PATH)
**Total LOC**: $TOTAL_LOC
**Languages**: ${LANGUAGES[@]}
**Frameworks**: ${FRAMEWORKS[@]}
**Strategy**: $ANALYSIS_STRATEGY

---

## Domain Findings

EOF

# Append each domain's findings
for domain in security performance concurrency jpa resilience architecture; do
    if [ -f "$OUTPUT_DIR/${domain}_findings.md" ]; then
        echo "### ${domain^} Domain" >> "$OUTPUT_DIR/CODE_REVIEW_REPORT_v3.0.md"
        echo "" >> "$OUTPUT_DIR/CODE_REVIEW_REPORT_v3.0.md"
        cat "$OUTPUT_DIR/${domain}_findings.md" >> "$OUTPUT_DIR/CODE_REVIEW_REPORT_v3.0.md"
        echo "" >> "$OUTPUT_DIR/CODE_REVIEW_REPORT_v3.0.md"
        echo "---" >> "$OUTPUT_DIR/CODE_REVIEW_REPORT_v3.0.md"
        echo "" >> "$OUTPUT_DIR/CODE_REVIEW_REPORT_v3.0.md"
    fi
done

echo "✓ Report assembled: $OUTPUT_DIR/CODE_REVIEW_REPORT_v3.0.md"
```

### Step 6.2: Generate Statistics

```bash
# Count findings by severity
CRITICAL=$(grep -oh "\*\*Severity\*\*: CRITICAL" "$OUTPUT_DIR/CODE_REVIEW_REPORT_v3.0.md" | wc -l)
HIGH=$(grep -oh "\*\*Severity\*\*: HIGH" "$OUTPUT_DIR/CODE_REVIEW_REPORT_v3.0.md" | wc -l)
MEDIUM=$(grep -oh "\*\*Severity\*\*: MEDIUM" "$OUTPUT_DIR/CODE_REVIEW_REPORT_v3.0.md" | wc -l)
LOW=$(grep -oh "\*\*Severity\*\*: LOW" "$OUTPUT_DIR/CODE_REVIEW_REPORT_v3.0.md" | wc -l)

cat >> "$OUTPUT_DIR/CODE_REVIEW_REPORT_v3.0.md" <<EOF

## Statistics

| Severity | Count |
|----------|-------|
| CRITICAL | $CRITICAL |
| HIGH | $HIGH |
| MEDIUM | $MEDIUM |
| LOW | $LOW |
| **TOTAL** | **$((CRITICAL + HIGH + MEDIUM + LOW))** |

---

## Analysis Metadata

- **Analysis Strategy**: $ANALYSIS_STRATEGY
- **Total LOC Analyzed**: $TOTAL_LOC
- **Hotspots Identified**: $HOTSPOT_COUNT
- **Framework Version**: 3.0
- **Completeness**: 100% (all findings documented)

---

**Report Generated**: $(date -u +%Y-%m-%dT%H:%M:%SZ)
**Framework**: Claude Code Review Framework v3.0

================================================================================
END OF REPORT
================================================================================

EOF

echo "✓ Statistics generated"
```

---

## POST-ANALYSIS TASKS

### Review Report

```bash
# Open report in editor
code "$OUTPUT_DIR/CODE_REVIEW_REPORT_v3.0.md"  # VS Code
# OR
open "$OUTPUT_DIR/CODE_REVIEW_REPORT_v3.0.md"  # macOS default
```

### Archive Analysis

```bash
# Create timestamped archive
ARCHIVE_NAME="code_review_$(date +%Y%m%d_%H%M%S).tar.gz"
tar -czf "$ARCHIVE_NAME" "$OUTPUT_DIR"
echo "✓ Analysis archived: $ARCHIVE_NAME"
```

---

## 🚨 TROUBLESHOOTING

### Issue: Agent returns "context overflow"

**Solution**:
1. Verify strategy selection (should be Progressive Writing for >100K LOC)
2. Re-run agent with explicit Progressive Writing instructions
3. Reduce scope if necessary (analyze critical modules first)

### Issue: Findings count mismatch (actual vs expected)

**Solution**:
1. Check agent output for `variance_reason` documentation
2. If undocumented: re-run agent with stricter validation
3. Review COMPLETENESS-ENFORCEMENT.md rules

### Issue: Summarization detected in output

**Solution**:
1. Re-run agent with explicit anti-summarization instructions
2. Reference COMPLETENESS-ENFORCEMENT.md section on forbidden phrases
3. Validate output using validation checklist

---

## 📊 SUCCESS METRICS

**Analysis is successful when**:

✅ All phases completed (0-6)
✅ All agents returned valid output
✅ No completeness violations detected
✅ All CRITICAL/HIGH findings documented in detail
✅ Quick Reference Table present for sampled findings
✅ Final report generated with statistics
✅ Context usage stayed <95% throughout

---

**Version**: 3.0 (AI-Optimized)
**Last Updated**: 2025-10-13

---

END OF ORCHESTRATOR TEMPLATE
