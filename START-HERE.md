# START HERE - Universal Code Review Framework Guide

**The definitive entry point for both AI models and human developers**

---

## 🚀 Quick Navigation

Choose your path based on your role and needs:

| You Are | Time Available | Start Here |
|---------|----------------|------------|
| **Human Developer** | 5 minutes | [→ 5-Minute Quick Scan](#5-minute-quick-scan) |
| **Human Developer** | 30+ minutes | [→ Full Analysis Workflow](#full-analysis-workflow) |
| **AI Model** | Any | [→ AI Model Instructions](#ai-model-instructions-mandatory) |
| **Looking for Examples** | Any | [→ Real-World Examples](#real-world-examples) |
| **Need Help** | Any | [→ Troubleshooting](#troubleshooting) |

---

## Part 1: 5-Minute Quick Scan (For Humans)

Get actionable insights in 5 minutes or less.

### Prerequisites
```bash
# Check you have required tools
command -v find && command -v grep && command -v wc || echo "Missing required tools"
```

### Quick Discovery Script
```bash
#!/bin/bash
# Save as: quick-scan.sh

echo "=== Repository Quick Scan ==="
echo "Date: $(date)"
echo ""

# Language detection
echo "=== Languages Detected ==="
find . -name "*.java" 2>/dev/null | head -1 && echo "✓ Java"
find . -name "*.py" 2>/dev/null | head -1 && echo "✓ Python"
find . -name "*.js" 2>/dev/null | head -1 && echo "✓ JavaScript"
find . -name "*.go" 2>/dev/null | head -1 && echo "✓ Go"

# Framework detection
echo ""
echo "=== Frameworks Detected ==="
test -f pom.xml && echo "✓ Maven (Java)"
test -f build.gradle && echo "✓ Gradle (Java)"
test -f requirements.txt && echo "✓ Python/pip"
test -f package.json && echo "✓ Node.js/npm"
test -f go.mod && echo "✓ Go modules"

# Size assessment
echo ""
echo "=== Codebase Size ==="
echo "Total files: $(find . -type f -name "*.java" -o -name "*.py" -o -name "*.js" 2>/dev/null | wc -l)"
echo "Total LOC: $(find . -type f \( -name "*.java" -o -name "*.py" -o -name "*.js" \) -exec wc -l {} + 2>/dev/null | tail -1 | awk '{print $1}')"

# Critical security issues
echo ""
echo "=== Security Hotspots (Top 5) ==="
echo "Hardcoded passwords:"
grep -r "password.*=.*['\"]" --include="*.java" --include="*.py" --include="*.js" 2>/dev/null | head -5

echo ""
echo "SQL injection risks:"
grep -r "query.*+" --include="*.java" --include="*.py" 2>/dev/null | head -5

# Performance issues
echo ""
echo "=== Performance Hotspots (Top 5) ==="
echo "N+1 query patterns:"
grep -r "for.*:" --include="*.java" --include="*.py" -A 2 2>/dev/null | grep -E "get[A-Z]|find|query" | head -5

echo ""
echo "=== Quick Scan Complete ==="
```

### Run It Now
```bash
chmod +x quick-scan.sh
./quick-scan.sh | tee quick-scan-results.txt
```

**What You Get in 5 Minutes**:
- Language and framework identification
- Codebase size assessment
- Top security vulnerabilities
- Major performance issues
- Actionable hotspots to investigate

---

## Part 2: AI Model Instructions (MANDATORY)

### ⚠️ STOP - Assessment Required

**Before proceeding, you MUST assess the codebase to choose the correct strategy.**

### Step 1: Codebase Assessment Checklist

Execute this assessment NOW:

```markdown
ASSESSMENT CHECKLIST:
□ Codebase size: _____ LOC (use find/wc commands)
□ Estimated findings: _____ issues (based on patterns found)
□ Available memory: Standard Claude context window
□ Output format needed: JSON / Markdown / Both
```

### Step 2: Strategy Selection (CRITICAL DECISION POINT)

Based on your assessment, choose your execution strategy:

| Codebase Size | Expected Findings | YOU MUST USE | Reason |
|---------------|-------------------|--------------|---------|
| **< 50K LOC** | < 100 issues | **[Standard Strategy](#standard-strategy)** | Fits in memory |
| **50-100K LOC** | 100-200 issues | **[Hybrid Strategy](#hybrid-strategy)** | May exceed limits |
| **> 100K LOC** | > 200 issues | **[Progressive Writing](#progressive-writing-strategy)** | Will overflow |

⚠️ **WARNING**: Choosing wrong strategy will cause:
- Standard on large codebase → 32K token overflow ERROR
- Progressive on small codebase → Unnecessary complexity

### Step 3: Mandatory Reading Order

Based on your chosen strategy, read these documents IN THIS EXACT ORDER:

#### Standard Strategy (<50K LOC)
1. **[COMPLETENESS-ENFORCEMENT.md](COMPLETENESS-ENFORCEMENT.md)** - 3-phase validation system
2. **[CLAUDE-ANALYSIS-FRAMEWORK.md](CLAUDE-ANALYSIS-FRAMEWORK.md)** - Core workflow
3. **[AGENT-PROMPTS.md](AGENT-PROMPTS.md)** - Agent templates
4. **You're done** - Proceed to execution

#### Hybrid Strategy (50-100K LOC)
1. **[UNIVERSAL-CONTEXT-MANAGEMENT.md](UNIVERSAL-CONTEXT-MANAGEMENT.md)** - Memory optimization
2. **[COMPLETENESS-ENFORCEMENT.md](COMPLETENESS-ENFORCEMENT.md)** - Validation system
3. **[CLAUDE-ANALYSIS-FRAMEWORK.md](CLAUDE-ANALYSIS-FRAMEWORK.md)** - Core workflow
4. **[AGENT-PROMPTS.md](AGENT-PROMPTS.md)** - Agent templates

#### Progressive Writing Strategy (>100K LOC)
1. **[UNIVERSAL-CONTEXT-MANAGEMENT.md](UNIVERSAL-CONTEXT-MANAGEMENT.md)** - Section 5.6 specifically
2. **[CLAUDE-ANALYSIS-FRAMEWORK.md](CLAUDE-ANALYSIS-FRAMEWORK.md)** - Progressive workflow
3. **[AGENT-PROMPTS.md](AGENT-PROMPTS.md)** - With write-clear pattern
4. **Initialize output files** before starting:
   ```bash
   touch security_findings.md performance_findings.md
   touch concurrency_findings.md architecture_findings.md
   ```

### Step 4: Execution Patterns by Strategy

#### Standard Strategy
```python
# Pattern: Accumulate in memory, return complete JSON
findings = []
for file in all_files:
    issues = analyze(file)
    findings.extend(issues)
return json.dumps(findings)  # All at once
```

#### Progressive Writing Strategy
```python
# Pattern: Write incrementally, clear memory, return summary only
with open('security_findings.md', 'a') as f:
    findings_batch = []
    for file in all_files:
        issues = analyze(file)
        findings_batch.extend(issues)

        if len(findings_batch) >= 50:  # Every 50 findings
            f.write(format_findings(findings_batch))
            findings_batch = []  # CLEAR MEMORY

    # Write remaining
    if findings_batch:
        f.write(format_findings(findings_batch))

# Return SUMMARY ONLY (not findings!)
return {
    "findings_written": 250,
    "file": "security_findings.md"
}
```

### Step 5: Critical Validation Rules

**ALL strategies must enforce**:

1. **No Summarization**: Never write "Found 8 SQL injections" without listing all 8
2. **Complete Listing**: Every finding must have file:line reference
3. **Validation Block**: Include `declared_count === actual_count` check
4. **Progress Reporting**: Report every 10% completion

### AI Model FAQ

**Q: What if I'm unsure about codebase size?**
A: Use Progressive Writing Strategy - it always works and never overflows.

**Q: Can I read all documents regardless of strategy?**
A: Yes, but follow the mandatory order for your chosen strategy.

**Q: What if analysis fails midway?**
A: With Progressive Writing, your findings are already saved to disk. With Standard, you lose everything.

**Q: Should I use multiple agents?**
A: Yes, run Security, Performance, Concurrency, and Architecture agents in parallel when possible.

---

## Part 3: Full Analysis Workflow

### Phase 0: Choose Your Execution Mode

```bash
# Run assessment first
echo "Checking codebase size..."
LOC=$(find . -name "*.java" -o -name "*.py" -o -name "*.js" | xargs wc -l | tail -1 | awk '{print $1}')

if [ "$LOC" -lt 50000 ]; then
    echo "→ Use STANDARD mode (in-memory analysis)"
    STRATEGY="standard"
elif [ "$LOC" -lt 100000 ]; then
    echo "→ Use HYBRID mode (compressed memory)"
    STRATEGY="hybrid"
else
    echo "→ Use PROGRESSIVE mode (write-clear-continue)"
    STRATEGY="progressive"
    # Initialize output files
    for category in security performance concurrency architecture; do
        echo "# $category Findings" > "${category}_findings.md"
        echo "Generated: $(date)" >> "${category}_findings.md"
        echo "---" >> "${category}_findings.md"
    done
fi
```

### Phase 1: Discovery (5 minutes)

```bash
#!/bin/bash
# discovery.sh - Complete project discovery

OUTPUT="discovery-manifest.json"

cat > $OUTPUT <<EOF
{
  "project": "$(basename $(pwd))",
  "analyzed_date": "$(date -Iseconds)",
  "languages": [],
  "frameworks": [],
  "statistics": {},
  "structure": {}
}
EOF

# Detect languages with file counts
echo "Detecting languages..."
for ext in java py js ts go rb php; do
    count=$(find . -name "*.$ext" 2>/dev/null | wc -l)
    if [ $count -gt 0 ]; then
        echo "  Found $count .$ext files"
        # Update JSON (simplified - use jq in practice)
    fi
done

# Detect frameworks
echo "Detecting frameworks..."
if [ -f pom.xml ]; then
    grep -q "spring-boot" pom.xml && echo "  ✓ Spring Boot detected"
    grep -q "hibernate" pom.xml && echo "  ✓ Hibernate detected"
fi

if [ -f requirements.txt ]; then
    grep -q "django" requirements.txt && echo "  ✓ Django detected"
    grep -q "flask" requirements.txt && echo "  ✓ Flask detected"
fi

if [ -f package.json ]; then
    grep -q "express" package.json && echo "  ✓ Express detected"
    grep -q "react" package.json && echo "  ✓ React detected"
fi

echo "Discovery complete. Manifest saved to $OUTPUT"
```

### Phase 2: Pattern Scanning (10 minutes)

```bash
#!/bin/bash
# pattern-scan.sh - Find hotspots quickly

echo "=== Scanning for Security Patterns ==="
echo ""

# SQL Injection
echo "[SQL_INJECTION]"
grep -r "query.*+" --include="*.java" --include="*.py" -n 2>/dev/null | \
    awk -F: '{print $1":"$2}' | sort -u > hotspots-sql-injection.txt
echo "Found $(wc -l < hotspots-sql-injection.txt) potential SQL injection points"

# Authentication gaps
echo ""
echo "[MISSING_AUTH]"
grep -r "@GetMapping\|@PostMapping\|@DeleteMapping" --include="*.java" -A 2 2>/dev/null | \
    grep -v "@PreAuthorize\|@Secured" | \
    grep -B 2 "public" > hotspots-auth.txt
echo "Found $(wc -l < hotspots-auth.txt) unprotected endpoints"

# N+1 Queries
echo ""
echo "[N_PLUS_ONE]"
grep -r "@OneToMany\|@ManyToOne" --include="*.java" -A 1 2>/dev/null | \
    grep -v "@BatchSize\|EAGER" > hotspots-n-plus-one.txt
echo "Found $(wc -l < hotspots-n-plus-one.txt) potential N+1 query patterns"

# Hardcoded secrets
echo ""
echo "[SECRETS]"
grep -r "password\|secret\|apikey\|token" --include="*.properties" --include="*.yml" \
    --include="*.env" 2>/dev/null | grep "=" > hotspots-secrets.txt
echo "Found $(wc -l < hotspots-secrets.txt) potential hardcoded secrets"

echo ""
echo "=== Pattern Scan Complete ==="
echo "Hotspot files generated for deep analysis"
```

### Phase 3: Agent Execution (30-60 minutes)

Based on your strategy, execute agents:

#### For Standard/Hybrid Strategy
```bash
# Run all agents in parallel (if possible)
echo "Launching analysis agents..."

# Security Agent
claude-code run-agent \
    --prompt security-agent-prompt.md \
    --files hotspots-sql-injection.txt,hotspots-auth.txt \
    --output findings-security.json &

# Performance Agent
claude-code run-agent \
    --prompt performance-agent-prompt.md \
    --files hotspots-n-plus-one.txt \
    --output findings-performance.json &

# Wait for completion
wait
echo "All agents complete"
```

#### For Progressive Writing Strategy
```bash
# Agents write to files incrementally
for agent in security performance concurrency architecture; do
    echo "Running $agent agent with progressive writing..."
    claude-code run-agent \
        --prompt ${agent}-agent-prompt.md \
        --progressive-write ${agent}_findings.md \
        --return-summary-only
done
```

### Phase 4: Assembly and Report Generation

```bash
#!/bin/bash
# generate-report.sh - Create final report

if [ "$STRATEGY" == "progressive" ]; then
    # Combine progressive output files
    cat > CODE_REVIEW_REPORT.md <<EOF
# Code Review Report
Generated: $(date)

## Summary
$(cat *_findings.md | grep -c "^###") total findings across all categories

## Findings by Category

EOF

    for file in *_findings.md; do
        cat $file >> CODE_REVIEW_REPORT.md
        echo "" >> CODE_REVIEW_REPORT.md
    done
else
    # Merge JSON findings
    jq -s 'add' findings-*.json > all-findings.json

    # Generate markdown report
    python3 generate-report.py all-findings.json > CODE_REVIEW_REPORT.md
fi

echo "Report generated: CODE_REVIEW_REPORT.md"
echo "Total findings: $(grep -c "^###" CODE_REVIEW_REPORT.md)"
```

---

## Part 4: Real-World Examples

### Example 1: Spring Boot Microservice (85K LOC)

**Discovery Output**:
```
=== Repository Statistics ===
Path: /home/user/payment-service
Language: Java (Spring Boot 3.2.0, Hibernate 6.2)
Total files: 523
Total LOC: 85,432
Modules: 12
→ Strategy: HYBRID (compressed memory)
```

**Security Scan Results**:
```
[SEC-CRIT-001] SQL Injection via String Concatenation
File: UserController.java:45
Pattern: String query = "SELECT * FROM users WHERE email = '" + email + "'";
Impact: Complete database compromise possible
Fix: Use PreparedStatement or @Query with parameters

[SEC-CRIT-002] Missing Authentication on Admin Endpoint
File: AdminController.java:89
Pattern: @DeleteMapping("/admin/users/{id}") // No @PreAuthorize!
Impact: Any user can delete other users
Fix: Add @PreAuthorize("hasRole('ADMIN')")

... 43 more security findings
```

**Performance Scan Results**:
```
[PERF-CRIT-001] Missing Hibernate Batch Configuration
File: application.yml
Impact: saveAll() operations execute as N individual INSERTs
Measurement: 50 entities take 2.5s instead of 0.1s
Fix: Add spring.jpa.properties.hibernate.jdbc.batch_size: 25

[PERF-CRIT-002] N+1 Query on User.orders Relationship
File: User.java:34
Pattern: @OneToMany(mappedBy = "user") // No @BatchSize!
Impact: Loading 100 users triggers 101 queries
Fix: Add @BatchSize(size = 10) or use JOIN FETCH
```

**Quick Wins Identified**:
```
1. Add batch configuration (30 min) → 25x faster bulk operations
2. Add @BatchSize to 15 relationships (2 hours) → 10x fewer queries
3. Add authentication to 5 endpoints (1 hour) → Critical security fix
Total effort: 3.5 hours
Expected improvement: 5-10x performance, 5 critical vulnerabilities fixed
```

### Example 2: Django E-commerce (45K LOC)

**Discovery Output**:
```
=== Repository Statistics ===
Path: /home/user/django-shop
Language: Python (Django 4.2, PostgreSQL)
Total files: 234
Total LOC: 45,123
Apps: 8 (users, products, orders, payments, etc.)
→ Strategy: STANDARD (in-memory)
```

**Critical Findings**:
```python
# [SEC-CRIT-001] SQL Injection in Raw Query
# File: views.py:234
cursor.execute(f"SELECT * FROM products WHERE category = '{category}'")
# Fix: Use parameterized query
cursor.execute("SELECT * FROM products WHERE category = %s", [category])

# [PERF-CRIT-001] N+1 Query in Order List View
# File: views.py:456
orders = Order.objects.all()
for order in orders:
    print(order.user.profile.name)  # N+1 query!
# Fix: Use select_related
orders = Order.objects.select_related('user__profile').all()
```

### Example 3: Node.js API (32K LOC)

**Discovery Output**:
```
=== Repository Statistics ===
Path: /home/user/api-gateway
Language: JavaScript (Express 4.18, MongoDB)
Total files: 156
Total LOC: 32,456
→ Strategy: STANDARD (in-memory)
```

**Critical Findings**:
```javascript
// [SEC-CRIT-001] XSS Vulnerability
// File: routes/user.js:45
app.get('/welcome', (req, res) => {
    res.send(`<h1>Welcome ${req.query.name}</h1>`); // XSS!
});
// Fix: Escape user input
res.send(`<h1>Welcome ${escape(req.query.name)}</h1>`);

// [PERF-CRIT-001] Synchronous File Operation Blocking Event Loop
// File: services/report.js:23
const data = fs.readFileSync('./large-file.json'); // BLOCKS!
// Fix: Use async version
const data = await fs.promises.readFile('./large-file.json');
```

---

## Part 5: Troubleshooting

### Common Issues and Solutions

#### Issue: "Token limit exceeded" error
**Diagnosis**: Codebase too large for chosen strategy
**Solution**:
```bash
# Switch to Progressive Writing Strategy immediately
# Initialize output files
for category in security performance concurrency architecture; do
    echo "# $category Findings" > "${category}_findings.md"
done
# Re-run with progressive writing enabled
```

#### Issue: "Analysis taking too long"
**Diagnosis**: Analyzing too many files without filtering
**Solution**:
```bash
# Use pattern scanning to identify hotspots first
grep -r "critical_pattern" --include="*.java" -l > hotspots.txt
# Analyze only hotspot files
claude-code analyze --files hotspots.txt
```

#### Issue: "Can't determine framework"
**Diagnosis**: Non-standard project structure
**Solution**:
```bash
# Create manual manifest
cat > manifest.json <<EOF
{
  "language": "Java",
  "framework": "Spring Boot",
  "database": "PostgreSQL"
}
EOF
# Pass to agents
claude-code analyze --context manifest.json
```

#### Issue: "Duplicate findings in report"
**Diagnosis**: Multiple agents finding same issue
**Solution**:
```python
# Deduplicate by hash
seen = set()
unique_findings = []
for finding in all_findings:
    hash_key = f"{finding['file']}:{finding['line']}:{finding['type']}"
    if hash_key not in seen:
        seen.add(hash_key)
        unique_findings.append(finding)
```

#### Issue: "Memory/Context overflow during analysis"
**Diagnosis**: Keeping too much in memory
**Solution**: Use the write-clear pattern
```python
# Wrong: Accumulate everything
all_findings = []
for file in files:
    all_findings.extend(analyze(file))  # Memory grows!

# Right: Write and clear
with open('output.md', 'a') as f:
    batch = []
    for file in files:
        batch.extend(analyze(file))
        if len(batch) >= 50:
            f.write(format_findings(batch))
            batch = []  # Clear memory!
```

---

## Next Steps

After reading this guide:

1. **For AI Models**: Proceed to your mandatory reading list based on strategy
2. **For Humans**: Run the 5-minute scan, then decide if you need deeper analysis
3. **For Examples**: See real-world results above or check LANGUAGE-PLUGINS.md
4. **For Scripts**: All scripts in this doc are ready to copy and run

## Quick Reference

| Action | Command/File |
|--------|-------------|
| Quick scan | `./quick-scan.sh` |
| Full discovery | `./discovery.sh` |
| Find hotspots | `./pattern-scan.sh` |
| Progressive setup | `touch {security,performance,concurrency,architecture}_findings.md` |
| Generate report | `./generate-report.sh` |
| Check strategy | `find . -name "*.{java,py,js}" \| xargs wc -l` |

## Document Index

| Document | Purpose | When to Read |
|----------|---------|--------------|
| **START-HERE.md** | This guide - entry point | Always first |
| **COMPLETENESS-ENFORCEMENT.md** | Anti-summarization rules | Before analysis |
| **UNIVERSAL-CONTEXT-MANAGEMENT.md** | Memory optimization | For large codebases |
| **CLAUDE-ANALYSIS-FRAMEWORK.md** | Core methodology | After strategy chosen |
| **AGENT-PROMPTS.md** | Agent templates | When running agents |
| **LANGUAGE-PLUGINS.md** | Language patterns | For specific languages |

---

## Version & Support

**Version**: 3.0 (Consolidated Edition)
**Last Updated**: 2024-10-12
**Framework Version**: 2.4

For additional examples and detailed language-specific patterns, see:
- [LANGUAGE-PLUGINS.md](LANGUAGE-PLUGINS.md) - Pattern catalog
- [SCRIPTS.md](SCRIPTS.md) - All scripts collection (if created)

**Remember**: When in doubt, use Progressive Writing Strategy - it always works!

---

END OF GUIDE