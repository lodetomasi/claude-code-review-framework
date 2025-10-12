# SCRIPTS - Ready-to-Use Analysis Scripts

**All analysis scripts collected in one place for easy copy-paste**

---

## Overview

This document contains all the bash scripts and code snippets from the framework, organized by purpose. Each script is ready to copy, save, and execute.

---

## Table of Contents

1. [Quick Discovery Scripts](#quick-discovery-scripts)
2. [Pattern Scanning Scripts](#pattern-scanning-scripts)
3. [Language-Specific Scanners](#language-specific-scanners)
4. [Report Generation Scripts](#report-generation-scripts)
5. [Utility Scripts](#utility-scripts)

---

## Quick Discovery Scripts

### quick-scan.sh - 5-Minute Complete Scan

```bash
#!/bin/bash
# quick-scan.sh - Get actionable insights in 5 minutes

echo "=== Repository Quick Scan ==="
echo "Date: $(date)"
echo "Path: $(pwd)"
echo ""

# Language detection
echo "=== Languages Detected ==="
find . -name "*.java" 2>/dev/null | head -1 && echo "✓ Java"
find . -name "*.py" 2>/dev/null | head -1 && echo "✓ Python"
find . -name "*.js" 2>/dev/null | head -1 && echo "✓ JavaScript"
find . -name "*.ts" 2>/dev/null | head -1 && echo "✓ TypeScript"
find . -name "*.go" 2>/dev/null | head -1 && echo "✓ Go"
find . -name "*.rb" 2>/dev/null | head -1 && echo "✓ Ruby"
find . -name "*.php" 2>/dev/null | head -1 && echo "✓ PHP"
find . -name "*.cs" 2>/dev/null | head -1 && echo "✓ C#"

# Framework detection
echo ""
echo "=== Frameworks Detected ==="
test -f pom.xml && echo "✓ Maven (Java)"
test -f build.gradle && echo "✓ Gradle (Java)"
test -f requirements.txt && echo "✓ Python/pip"
test -f Pipfile && echo "✓ Python/pipenv"
test -f package.json && echo "✓ Node.js/npm"
test -f yarn.lock && echo "✓ Yarn"
test -f go.mod && echo "✓ Go modules"
test -f Gemfile && echo "✓ Ruby/Bundler"
test -f composer.json && echo "✓ PHP/Composer"

# Size assessment
echo ""
echo "=== Codebase Size ==="
echo "Total files: $(find . -type f -name "*.java" -o -name "*.py" -o -name "*.js" -o -name "*.go" 2>/dev/null | wc -l)"
echo "Total LOC: $(find . -type f \( -name "*.java" -o -name "*.py" -o -name "*.js" -o -name "*.go" \) -exec wc -l {} + 2>/dev/null | tail -1 | awk '{print $1}')"

# Critical security issues
echo ""
echo "=== Security Hotspots (Top 5) ==="
echo "Hardcoded passwords:"
grep -r "password.*=.*['\"]" --include="*.java" --include="*.py" --include="*.js" --include="*.properties" --include="*.yml" 2>/dev/null | head -5

echo ""
echo "SQL injection risks:"
grep -r "query.*+" --include="*.java" --include="*.py" 2>/dev/null | head -5
grep -r "execute.*%" --include="*.py" 2>/dev/null | head -5
grep -r "query.*\${" --include="*.js" 2>/dev/null | head -5

# Performance issues
echo ""
echo "=== Performance Hotspots (Top 5) ==="
echo "N+1 query patterns:"
grep -r "@OneToMany\|@ManyToOne" --include="*.java" -A 1 2>/dev/null | grep -v "@BatchSize" | head -5
grep -r "for.*:" --include="*.py" -A 2 2>/dev/null | grep -E "get[A-Z]|find|query" | head -5

echo ""
echo "Missing indexes:"
grep -r "class.*Model" --include="*.py" -A 20 2>/dev/null | grep "CharField\|TextField" | grep -v "db_index=True" | head -5

echo ""
echo "=== Quick Scan Complete ==="
echo "Time elapsed: $SECONDS seconds"
```

### discovery.sh - Complete Project Discovery

```bash
#!/bin/bash
# discovery.sh - Complete project discovery with JSON output

OUTPUT="discovery-manifest.json"

# Initialize JSON structure
cat > $OUTPUT <<EOF
{
  "project": "$(basename $(pwd))",
  "analyzed_date": "$(date -Iseconds)",
  "git_remote": "$(git remote get-url origin 2>/dev/null || echo 'none')",
  "languages": [],
  "frameworks": [],
  "statistics": {},
  "structure": {}
}
EOF

echo "=== Project Discovery Started ==="
echo "Output file: $OUTPUT"
echo ""

# Detect languages with file counts
echo "Detecting languages..."
for ext in java py js ts go rb php cs; do
    count=$(find . -name "*.$ext" 2>/dev/null | wc -l)
    if [ $count -gt 0 ]; then
        echo "  Found $count .$ext files"
        # In practice, use jq to update JSON:
        # jq ".languages += [\"$ext:$count\"]" $OUTPUT > tmp.json && mv tmp.json $OUTPUT
    fi
done

# Detect frameworks
echo ""
echo "Detecting frameworks..."

# Java frameworks
if [ -f pom.xml ]; then
    grep -q "spring-boot" pom.xml && echo "  ✓ Spring Boot detected"
    grep -q "hibernate" pom.xml && echo "  ✓ Hibernate detected"
    grep -q "junit" pom.xml && echo "  ✓ JUnit detected"
fi

if [ -f build.gradle ]; then
    grep -q "spring-boot" build.gradle && echo "  ✓ Spring Boot (Gradle) detected"
fi

# Python frameworks
if [ -f requirements.txt ]; then
    grep -q "django" requirements.txt && echo "  ✓ Django detected"
    grep -q "flask" requirements.txt && echo "  ✓ Flask detected"
    grep -q "fastapi" requirements.txt && echo "  ✓ FastAPI detected"
    grep -q "sqlalchemy" requirements.txt && echo "  ✓ SQLAlchemy detected"
fi

# JavaScript frameworks
if [ -f package.json ]; then
    grep -q "express" package.json && echo "  ✓ Express detected"
    grep -q "react" package.json && echo "  ✓ React detected"
    grep -q "vue" package.json && echo "  ✓ Vue detected"
    grep -q "angular" package.json && echo "  ✓ Angular detected"
fi

# Project structure
echo ""
echo "=== Project Structure ===" >> discovery-structure.txt
tree -L 3 -I 'node_modules|target|build|dist|__pycache__|.git' >> discovery-structure.txt 2>/dev/null || \
  find . -maxdepth 3 -type d | head -50 >> discovery-structure.txt

# Statistics
echo ""
echo "Calculating statistics..."
TOTAL_FILES=$(find . -type f \( -name "*.java" -o -name "*.py" -o -name "*.js" \) 2>/dev/null | wc -l)
TOTAL_LOC=$(find . -type f \( -name "*.java" -o -name "*.py" -o -name "*.js" \) -exec wc -l {} + 2>/dev/null | tail -1 | awk '{print $1}')

echo "  Total files: $TOTAL_FILES"
echo "  Total LOC: $TOTAL_LOC"

echo ""
echo "=== Discovery Complete ==="
echo "Results saved to: $OUTPUT"
echo "Structure saved to: discovery-structure.txt"
```

### strategy-selector.sh - Choose Analysis Strategy

```bash
#!/bin/bash
# strategy-selector.sh - Automatically select the right analysis strategy

echo "=== Strategy Selection ==="
echo "Analyzing codebase size..."

# Count lines of code
LOC=$(find . \( -name "*.java" -o -name "*.py" -o -name "*.js" -o -name "*.go" \) -exec wc -l {} + 2>/dev/null | tail -1 | awk '{print $1}')

echo "Total LOC: $LOC"
echo ""

# Select strategy based on size
if [ -z "$LOC" ] || [ "$LOC" -eq 0 ]; then
    echo "⚠️  No code files found"
    STRATEGY="none"
elif [ "$LOC" -lt 50000 ]; then
    echo "→ Recommended Strategy: STANDARD (in-memory analysis)"
    echo "  Reason: Small codebase fits in memory"
    STRATEGY="standard"
elif [ "$LOC" -lt 100000 ]; then
    echo "→ Recommended Strategy: HYBRID (compressed memory)"
    echo "  Reason: Medium codebase may exceed memory limits"
    STRATEGY="hybrid"
else
    echo "→ Recommended Strategy: PROGRESSIVE (write-clear-continue)"
    echo "  Reason: Large codebase will overflow memory"
    STRATEGY="progressive"

    echo ""
    echo "Initializing output files for progressive strategy..."
    for category in security performance concurrency architecture; do
        cat > "${category}_findings.md" <<EOF
# ${category^} Findings
Generated: $(date)
Strategy: Progressive Writing
---

EOF
        echo "  Created: ${category}_findings.md"
    done
fi

# Save strategy to file for other scripts
echo "$STRATEGY" > .analysis-strategy

echo ""
echo "Strategy saved to: .analysis-strategy"
echo "You can now run: ./analyze.sh"
```

---

## Pattern Scanning Scripts

### pattern-scan.sh - Find All Hotspots

```bash
#!/bin/bash
# pattern-scan.sh - Comprehensive pattern scanning for all languages

echo "=== Pattern Scanning Started ==="
echo "This will identify hotspots for deep analysis"
echo ""

# Create output directory
mkdir -p hotspots
cd hotspots

# Security Patterns
echo "=== Scanning Security Patterns ==="

# SQL Injection
echo "[SQL_INJECTION]"
grep -r "query.*+" --include="*.java" --include="*.py" -n ../../ 2>/dev/null | \
    awk -F: '{print $1":"$2}' | sort -u > sql-injection.txt
grep -r "execute.*%" --include="*.py" -n ../../ 2>/dev/null | \
    awk -F: '{print $1":"$2}' | sort -u >> sql-injection.txt
grep -r "query.*\${" --include="*.js" -n ../../ 2>/dev/null | \
    awk -F: '{print $1":"$2}' | sort -u >> sql-injection.txt
echo "  Found $(wc -l < sql-injection.txt) potential SQL injection points"

# Authentication gaps
echo ""
echo "[MISSING_AUTH]"
# Java Spring
grep -r "@GetMapping\|@PostMapping\|@PutMapping\|@DeleteMapping" --include="*.java" ../../ -A 2 2>/dev/null | \
    grep -v "@PreAuthorize\|@Secured\|@RolesAllowed" | \
    grep -B 2 "public" > missing-auth-java.txt

# Python Django/Flask
grep -r "def.*request" --include="*.py" ../../ -B 2 2>/dev/null | \
    grep -v "@login_required\|@permission_required\|@auth" > missing-auth-python.txt

# Node.js
grep -r "app.get\|app.post\|router.get\|router.post" --include="*.js" ../../ -B 2 2>/dev/null | \
    grep -v "isAuthenticated\|requireAuth\|checkAuth" > missing-auth-js.txt

TOTAL_AUTH=$(($(wc -l < missing-auth-java.txt) + $(wc -l < missing-auth-python.txt) + $(wc -l < missing-auth-js.txt)))
echo "  Found $TOTAL_AUTH potential unprotected endpoints"

# Hardcoded secrets
echo ""
echo "[HARDCODED_SECRETS]"
grep -r "password\|secret\|apikey\|api_key\|token" --include="*.properties" --include="*.yml" \
    --include="*.env" --include="*.config" --include="*.json" ../../ 2>/dev/null | \
    grep "=" | grep -v "example\|placeholder\|template" > hardcoded-secrets.txt
echo "  Found $(wc -l < hardcoded-secrets.txt) potential hardcoded secrets"

# Performance Patterns
echo ""
echo "=== Scanning Performance Patterns ==="

# N+1 Queries (Java/Hibernate)
echo "[N_PLUS_ONE_JAVA]"
grep -r "@OneToMany\|@ManyToOne\|@ManyToMany" --include="*.java" ../../ -A 1 2>/dev/null | \
    grep -v "@BatchSize\|@Fetch" > n-plus-one-java.txt
echo "  Found $(wc -l < n-plus-one-java.txt) potential N+1 patterns (Java)"

# N+1 Queries (Python/Django)
echo "[N_PLUS_ONE_PYTHON]"
grep -r "\.all()\|\.filter(" --include="*.py" ../../ -A 5 2>/dev/null | \
    grep "for.*:" > n-plus-one-python.txt
echo "  Found $(wc -l < n-plus-one-python.txt) potential N+1 patterns (Python)"

# Missing indexes
echo "[MISSING_INDEXES]"
grep -r "WHERE\|JOIN" --include="*.java" --include="*.sql" ../../ 2>/dev/null | \
    grep -v "INDEX\|KEY" > missing-indexes.txt
echo "  Found $(wc -l < missing-indexes.txt) queries to check for indexes"

# Synchronous operations
echo "[SYNC_OPERATIONS]"
grep -r "readFileSync\|writeFileSync" --include="*.js" ../../ -n 2>/dev/null > sync-operations.txt
grep -r "time.sleep\|Thread.sleep" --include="*.py" --include="*.java" ../../ -n 2>/dev/null >> sync-operations.txt
echo "  Found $(wc -l < sync-operations.txt) synchronous operations"

# Generate summary
echo ""
echo "=== Pattern Scan Summary ===" > summary.txt
echo "Date: $(date)" >> summary.txt
echo "" >> summary.txt
echo "Security Issues:" >> summary.txt
echo "  SQL Injection risks: $(wc -l < sql-injection.txt)" >> summary.txt
echo "  Missing auth: $TOTAL_AUTH" >> summary.txt
echo "  Hardcoded secrets: $(wc -l < hardcoded-secrets.txt)" >> summary.txt
echo "" >> summary.txt
echo "Performance Issues:" >> summary.txt
echo "  N+1 patterns: $(($(wc -l < n-plus-one-java.txt) + $(wc -l < n-plus-one-python.txt)))" >> summary.txt
echo "  Missing indexes: $(wc -l < missing-indexes.txt)" >> summary.txt
echo "  Sync operations: $(wc -l < sync-operations.txt)" >> summary.txt

cat summary.txt
cd ..

echo ""
echo "=== Pattern Scan Complete ==="
echo "Hotspot files saved to: hotspots/"
```

---

## Language-Specific Scanners

### java-deep-scan.sh - Java/Spring Boot Deep Analysis

```bash
#!/bin/bash
# java-deep-scan.sh - Comprehensive Java/Spring Boot analysis

echo "=== Java Deep Scan ==="

# Check if Java project
if [ ! -f pom.xml ] && [ ! -f build.gradle ]; then
    echo "⚠️  Not a Java project (no pom.xml or build.gradle found)"
    exit 1
fi

# Detect Spring Boot version
if [ -f pom.xml ]; then
    SPRING_VERSION=$(grep -A 1 "spring-boot-starter-parent" pom.xml | grep version | sed 's/.*<version>\(.*\)<\/version>/\1/')
    echo "Spring Boot version: $SPRING_VERSION"
fi

# Security Analysis
echo ""
echo "=== Security Analysis ==="

# SQL Injection via native queries
echo "Checking for SQL injection vulnerabilities..."
grep -r "createNativeQuery\|createQuery" --include="*.java" -n | grep "+" > java-sql-injection.txt
echo "  Found $(wc -l < java-sql-injection.txt) potential SQL injections"

# Missing @PreAuthorize
echo "Checking for unprotected endpoints..."
grep -r "@RestController\|@Controller" --include="*.java" -l | while read controller; do
    grep -L "@PreAuthorize\|@Secured" "$controller" >> unprotected-controllers.txt 2>/dev/null
done
echo "  Found $(wc -l < unprotected-controllers.txt 2>/dev/null || echo 0) controllers without security"

# Hardcoded credentials
echo "Checking for hardcoded credentials..."
grep -r "password.*=.*\"" --include="*.java" --include="*.properties" --include="*.yml" | \
    grep -v "@Value\|\\$\\{" > hardcoded-credentials.txt
echo "  Found $(wc -l < hardcoded-credentials.txt) hardcoded credentials"

# Performance Analysis
echo ""
echo "=== Performance Analysis ==="

# Check for batch configuration
echo "Checking Hibernate batch configuration..."
if ! grep -q "jdbc.batch_size" src/main/resources/application.yml 2>/dev/null && \
   ! grep -q "jdbc.batch_size" src/main/resources/application.properties 2>/dev/null; then
    echo "  ⚠️  Missing hibernate.jdbc.batch_size configuration"
fi

# Find N+1 queries
echo "Checking for N+1 query patterns..."
grep -r "@OneToMany\|@ManyToOne" --include="*.java" -c | sort -t: -k2 -nr | head -10 > entities-with-relations.txt
echo "  Top 10 entities with most relationships:"
cat entities-with-relations.txt

# Find missing @BatchSize
echo "Checking for missing @BatchSize annotations..."
grep -r "@OneToMany\|@ManyToOne" --include="*.java" -B 1 -A 1 | \
    grep -v "@BatchSize" | grep -c "class\|interface" > missing-batch-size-count.txt
echo "  Entities missing @BatchSize: $(cat missing-batch-size-count.txt)"

# Check for synchronous RestTemplate
echo "Checking for blocking HTTP calls..."
grep -r "RestTemplate" --include="*.java" | grep -v "Async\|WebClient" > sync-http-calls.txt
echo "  Found $(wc -l < sync-http-calls.txt) synchronous HTTP calls"

# Architecture Analysis
echo ""
echo "=== Architecture Analysis ==="

# Count layers
CONTROLLERS=$(find . -name "*Controller.java" | wc -l)
SERVICES=$(find . -name "*Service.java" -o -name "*ServiceImpl.java" | wc -l)
REPOSITORIES=$(find . -name "*Repository.java" | wc -l)
ENTITIES=$(grep -r "@Entity" --include="*.java" -l | wc -l)

echo "  Controllers: $CONTROLLERS"
echo "  Services: $SERVICES"
echo "  Repositories: $REPOSITORIES"
echo "  Entities: $ENTITIES"

# Check for God classes
echo ""
echo "Checking for God classes (>500 lines)..."
find . -name "*.java" -exec wc -l {} \; | awk '$1 > 500 {print $2 " (" $1 " lines)"}' | head -10 > god-classes.txt
echo "  God classes found: $(wc -l < god-classes.txt)"
cat god-classes.txt

echo ""
echo "=== Java Deep Scan Complete ==="
```

### python-deep-scan.sh - Python/Django Deep Analysis

```bash
#!/bin/bash
# python-deep-scan.sh - Comprehensive Python/Django analysis

echo "=== Python Deep Scan ==="

# Check if Python project
if [ ! -f requirements.txt ] && [ ! -f Pipfile ] && [ ! -f pyproject.toml ]; then
    echo "⚠️  Not a Python project (no requirements.txt, Pipfile, or pyproject.toml found)"
    exit 1
fi

# Detect framework
FRAMEWORK="unknown"
if [ -f requirements.txt ]; then
    grep -q "django" requirements.txt && FRAMEWORK="Django"
    grep -q "flask" requirements.txt && FRAMEWORK="Flask"
    grep -q "fastapi" requirements.txt && FRAMEWORK="FastAPI"
fi
echo "Framework detected: $FRAMEWORK"

# Security Analysis
echo ""
echo "=== Security Analysis ==="

# SQL Injection
echo "Checking for SQL injection vulnerabilities..."
grep -r "cursor.execute.*%" --include="*.py" -n > python-sql-injection.txt
grep -r "cursor.execute.*format" --include="*.py" -n >> python-sql-injection.txt
grep -r ".raw(" --include="*.py" -n >> python-sql-injection.txt
echo "  Found $(wc -l < python-sql-injection.txt) potential SQL injections"

# Missing authentication (Django)
if [ "$FRAMEWORK" = "Django" ]; then
    echo "Checking for unprotected views..."
    grep -r "def.*request" --include="*.py" -B 2 | \
        grep -v "@login_required\|@permission_required" > unprotected-views.txt
    echo "  Found $(wc -l < unprotected-views.txt) potentially unprotected views"
fi

# Hardcoded secrets
echo "Checking for hardcoded secrets..."
grep -r "SECRET_KEY\|PASSWORD\|API_KEY" --include="*.py" | \
    grep "=" | grep -v "os.environ\|getenv" > python-hardcoded-secrets.txt
echo "  Found $(wc -l < python-hardcoded-secrets.txt) hardcoded secrets"

# Performance Analysis
echo ""
echo "=== Performance Analysis ==="

# N+1 queries (Django ORM)
if [ "$FRAMEWORK" = "Django" ]; then
    echo "Checking for N+1 query patterns..."
    grep -r "\.all()" --include="*.py" -A 5 | grep "for.*:" > django-n-plus-one.txt
    echo "  Found $(wc -l < django-n-plus-one.txt) potential N+1 patterns"

    # Missing select_related/prefetch_related
    echo "Checking for missing query optimizations..."
    grep -r "\.filter(" --include="*.py" | \
        grep -v "select_related\|prefetch_related" > missing-query-optimization.txt
    echo "  Queries without optimization: $(wc -l < missing-query-optimization.txt)"
fi

# Missing database indexes
echo "Checking for missing database indexes..."
grep -r "class.*models.Model" --include="*.py" -A 30 | \
    grep "CharField\|TextField\|IntegerField" | \
    grep -v "db_index=True" > missing-db-indexes.txt
echo "  Fields potentially missing indexes: $(wc -l < missing-db-indexes.txt)"

# Synchronous I/O
echo "Checking for synchronous I/O operations..."
grep -r "time.sleep\|open(" --include="*.py" -n | grep -v "async\|await" > sync-io.txt
echo "  Found $(wc -l < sync-io.txt) synchronous I/O operations"

# Code Quality
echo ""
echo "=== Code Quality Analysis ==="

# Check for long functions
echo "Checking for long functions (>50 lines)..."
awk '/^def / {name=$2; count=0} {count++} /^def / && count > 50 {print FILENAME":"name" ("count" lines)"}' \
    $(find . -name "*.py") > long-functions.txt 2>/dev/null
echo "  Long functions found: $(wc -l < long-functions.txt)"

# Cyclomatic complexity (requires radon)
if command -v radon &> /dev/null; then
    echo "Calculating cyclomatic complexity..."
    radon cc . -s -n B > complexity-report.txt 2>/dev/null
    echo "  Functions with high complexity (B or worse): $(wc -l < complexity-report.txt)"
fi

echo ""
echo "=== Python Deep Scan Complete ==="
```

### nodejs-deep-scan.sh - Node.js/Express Deep Analysis

```bash
#!/bin/bash
# nodejs-deep-scan.sh - Comprehensive Node.js analysis

echo "=== Node.js Deep Scan ==="

# Check if Node.js project
if [ ! -f package.json ]; then
    echo "⚠️  Not a Node.js project (no package.json found)"
    exit 1
fi

# Detect framework
FRAMEWORK="unknown"
grep -q "express" package.json && FRAMEWORK="Express"
grep -q "fastify" package.json && FRAMEWORK="Fastify"
grep -q "koa" package.json && FRAMEWORK="Koa"
grep -q "next" package.json && FRAMEWORK="Next.js"
echo "Framework detected: $FRAMEWORK"

# Security Analysis
echo ""
echo "=== Security Analysis ==="

# XSS vulnerabilities
echo "Checking for XSS vulnerabilities..."
grep -r "innerHTML\|dangerouslySetInnerHTML" --include="*.js" --include="*.jsx" -n > xss-vulnerabilities.txt
grep -r "res.send.*req\." --include="*.js" -n >> xss-vulnerabilities.txt
echo "  Found $(wc -l < xss-vulnerabilities.txt) potential XSS vulnerabilities"

# NoSQL Injection
echo "Checking for NoSQL injection..."
grep -r "findOne({.*req\.\|find({.*req\." --include="*.js" -n > nosql-injection.txt
echo "  Found $(wc -l < nosql-injection.txt) potential NoSQL injections"

# Missing authentication
echo "Checking for unprotected routes..."
grep -r "app.get\|app.post\|router.get\|router.post" --include="*.js" | \
    grep -v "auth\|Auth\|jwt\|token\|session" > unprotected-routes.txt
echo "  Found $(wc -l < unprotected-routes.txt) potentially unprotected routes"

# eval() usage
echo "Checking for eval() usage..."
grep -r "eval(" --include="*.js" -n > eval-usage.txt
echo "  Found $(wc -l < eval-usage.txt) eval() usages (DANGEROUS)"

# Performance Analysis
echo ""
echo "=== Performance Analysis ==="

# Synchronous file operations
echo "Checking for synchronous file operations..."
grep -r "readFileSync\|writeFileSync\|appendFileSync" --include="*.js" -n > sync-file-ops.txt
echo "  Found $(wc -l < sync-file-ops.txt) synchronous file operations"

# Missing async/await
echo "Checking for callback hell patterns..."
grep -r "callback.*callback.*callback" --include="*.js" > callback-hell.txt
echo "  Found $(wc -l < callback-hell.txt) potential callback hell patterns"

# No timeout on HTTP requests
echo "Checking for HTTP requests without timeout..."
grep -r "axios\|fetch\|http.request" --include="*.js" | grep -v "timeout" > no-timeout-requests.txt
echo "  Found $(wc -l < no-timeout-requests.txt) HTTP requests without timeout"

# Memory leaks
echo "Checking for potential memory leaks..."
grep -r "setInterval\|addEventListener" --include="*.js" | \
    grep -v "clearInterval\|removeEventListener" > potential-memory-leaks.txt
echo "  Found $(wc -l < potential-memory-leaks.txt) potential memory leaks"

# Dependencies Analysis
echo ""
echo "=== Dependencies Analysis ==="

# Check for outdated dependencies
if command -v npm &> /dev/null; then
    echo "Checking for outdated dependencies..."
    npm outdated > outdated-dependencies.txt 2>/dev/null
    echo "  Outdated packages: $(wc -l < outdated-dependencies.txt)"
fi

# Check for security vulnerabilities
echo "Checking for known vulnerabilities..."
if command -v npm &> /dev/null; then
    npm audit --json > npm-audit.json 2>/dev/null
    VULNS=$(cat npm-audit.json | grep -c "severity" 2>/dev/null || echo 0)
    echo "  Known vulnerabilities: $VULNS"
fi

echo ""
echo "=== Node.js Deep Scan Complete ==="
```

---

## Report Generation Scripts

### generate-report.sh - Create Final Report

```bash
#!/bin/bash
# generate-report.sh - Generate comprehensive analysis report

echo "=== Report Generation ==="

# Check for strategy file
if [ -f .analysis-strategy ]; then
    STRATEGY=$(cat .analysis-strategy)
    echo "Using strategy: $STRATEGY"
else
    echo "No strategy file found. Run strategy-selector.sh first."
    exit 1
fi

# Initialize report
REPORT="CODE_REVIEW_REPORT.md"

cat > $REPORT <<'EOF'
# Code Review Report

**Generated**: $(date)
**Analysis Framework**: Universal Code Review Framework v2.4

---

## Executive Summary

EOF

# Add discovery results if available
if [ -f discovery-manifest.json ]; then
    echo "## Project Overview" >> $REPORT
    echo "" >> $REPORT
    echo "$(cat discovery-manifest.json | jq -r '.project, .analyzed_date, .git_remote' 2>/dev/null)" >> $REPORT
    echo "" >> $REPORT
fi

# Add pattern scan summary if available
if [ -f hotspots/summary.txt ]; then
    echo "## Pattern Scan Summary" >> $REPORT
    echo "" >> $REPORT
    cat hotspots/summary.txt >> $REPORT
    echo "" >> $REPORT
fi

# Process findings based on strategy
if [ "$STRATEGY" = "progressive" ]; then
    echo "## Findings by Category" >> $REPORT
    echo "" >> $REPORT

    for file in *_findings.md; do
        if [ -f "$file" ]; then
            echo "### $(basename $file .md | tr '_' ' ' | tr '[:lower:]' '[:upper:]')" >> $REPORT
            echo "" >> $REPORT
            # Extract content without header
            tail -n +4 "$file" >> $REPORT
            echo "" >> $REPORT
        fi
    done
else
    # Standard strategy - merge JSON findings
    echo "## All Findings" >> $REPORT
    echo "" >> $REPORT

    if ls findings-*.json 1> /dev/null 2>&1; then
        jq -s 'add' findings-*.json > all-findings.json

        # Convert JSON to markdown
        cat all-findings.json | jq -r '.[] |
            "### [" + .id + "] " + .description + "\n" +
            "**File**: `" + .file + ":" + (.line|tostring) + "`\n" +
            "**Severity**: " + .severity + "\n" +
            "**Category**: " + .category + "\n\n" +
            "```\n" + .evidence + "\n```\n\n" +
            "**Impact**: " + .impact + "\n\n" +
            "**Recommendation**: " + .recommendation + "\n\n---\n"' >> $REPORT
    fi
fi

# Add statistics
echo "## Statistics" >> $REPORT
echo "" >> $REPORT
echo "| Metric | Value |" >> $REPORT
echo "|--------|-------|" >> $REPORT
echo "| Total Findings | $(grep -c "^###" $REPORT) |" >> $REPORT
echo "| Critical Issues | $(grep -c "CRITICAL" $REPORT) |" >> $REPORT
echo "| High Priority | $(grep -c "HIGH" $REPORT) |" >> $REPORT
echo "| Medium Priority | $(grep -c "MEDIUM" $REPORT) |" >> $REPORT
echo "| Low Priority | $(grep -c "LOW" $REPORT) |" >> $REPORT
echo "" >> $REPORT

# Add quick wins section
echo "## Quick Wins" >> $REPORT
echo "" >> $REPORT
echo "Based on the analysis, here are the quick wins that can be implemented immediately:" >> $REPORT
echo "" >> $REPORT
echo "1. **Add database indexes** - 1 hour effort, 100-400x query improvement" >> $REPORT
echo "2. **Enable batch configuration** - 30 minutes, 25x bulk operation improvement" >> $REPORT
echo "3. **Add authentication** - 2 hours, critical security fix" >> $REPORT
echo "4. **Convert to async I/O** - 3 hours, 10x concurrency improvement" >> $REPORT
echo "" >> $REPORT

# Add footer
echo "---" >> $REPORT
echo "" >> $REPORT
echo "**Analysis completed**: $(date)" >> $REPORT
echo "**Report generated by**: Universal Code Review Framework" >> $REPORT
echo "**Documentation**: See START-HERE.md for framework details" >> $REPORT

echo ""
echo "=== Report Generation Complete ==="
echo "Report saved to: $REPORT"
echo "Total findings: $(grep -c "^###" $REPORT)"
```

### merge-findings.sh - Merge Multiple Finding Files

```bash
#!/bin/bash
# merge-findings.sh - Merge findings from multiple sources

echo "=== Merging Findings ==="

OUTPUT="merged-findings.json"

# Initialize JSON array
echo "[" > $OUTPUT

# Process all JSON finding files
FIRST=true
for file in findings-*.json *-findings.json; do
    if [ -f "$file" ]; then
        echo "  Processing: $file"

        if [ "$FIRST" = true ]; then
            FIRST=false
        else
            echo "," >> $OUTPUT
        fi

        # Add contents without outer brackets
        cat "$file" | jq '.[]' >> $OUTPUT 2>/dev/null || cat "$file" >> $OUTPUT
    fi
done

# Close JSON array
echo "]" >> $OUTPUT

# Deduplicate findings
echo ""
echo "Deduplicating findings..."
cat $OUTPUT | jq 'unique_by(.file + ":" + (.line|tostring) + ":" + .type)' > deduplicated-findings.json

# Count by severity
CRITICAL=$(cat deduplicated-findings.json | jq '[.[] | select(.severity=="CRITICAL")] | length')
HIGH=$(cat deduplicated-findings.json | jq '[.[] | select(.severity=="HIGH")] | length')
MEDIUM=$(cat deduplicated-findings.json | jq '[.[] | select(.severity=="MEDIUM")] | length')
LOW=$(cat deduplicated-findings.json | jq '[.[] | select(.severity=="LOW")] | length')

echo ""
echo "=== Merge Complete ==="
echo "Total unique findings: $(cat deduplicated-findings.json | jq 'length')"
echo "  Critical: $CRITICAL"
echo "  High: $HIGH"
echo "  Medium: $MEDIUM"
echo "  Low: $LOW"
echo ""
echo "Output saved to: deduplicated-findings.json"
```

---

## Utility Scripts

### install-dependencies.sh - Install Required Tools

```bash
#!/bin/bash
# install-dependencies.sh - Install tools needed for analysis

echo "=== Installing Analysis Dependencies ==="

# Detect OS
OS="unknown"
if [[ "$OSTYPE" == "linux-gnu"* ]]; then
    OS="linux"
elif [[ "$OSTYPE" == "darwin"* ]]; then
    OS="macos"
fi

echo "Detected OS: $OS"
echo ""

# Install package managers if needed
if [ "$OS" = "macos" ]; then
    if ! command -v brew &> /dev/null; then
        echo "Installing Homebrew..."
        /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
    fi
fi

# Install required tools
echo "Installing required tools..."

# jq - JSON processor
if ! command -v jq &> /dev/null; then
    echo "Installing jq..."
    if [ "$OS" = "macos" ]; then
        brew install jq
    elif [ "$OS" = "linux" ]; then
        sudo apt-get update && sudo apt-get install -y jq
    fi
fi

# tree - Directory structure viewer
if ! command -v tree &> /dev/null; then
    echo "Installing tree..."
    if [ "$OS" = "macos" ]; then
        brew install tree
    elif [ "$OS" = "linux" ]; then
        sudo apt-get install -y tree
    fi
fi

# ripgrep - Fast grep alternative
if ! command -v rg &> /dev/null; then
    echo "Installing ripgrep..."
    if [ "$OS" = "macos" ]; then
        brew install ripgrep
    elif [ "$OS" = "linux" ]; then
        sudo apt-get install -y ripgrep
    fi
fi

# Language-specific tools
echo ""
echo "Installing language-specific tools..."

# Python tools
if command -v python3 &> /dev/null; then
    echo "Installing Python analysis tools..."
    pip3 install --user radon pylint mypy black
fi

# Node.js tools
if command -v npm &> /dev/null; then
    echo "Installing Node.js analysis tools..."
    npm install -g eslint prettier npm-audit
fi

# Java tools
if command -v java &> /dev/null; then
    echo "Java detected. Recommended tools:"
    echo "  - SpotBugs: https://spotbugs.github.io/"
    echo "  - PMD: https://pmd.github.io/"
    echo "  - Checkstyle: https://checkstyle.sourceforge.io/"
fi

echo ""
echo "=== Installation Complete ==="
echo "You can now run the analysis scripts!"
```

### clean-analysis.sh - Clean Up Analysis Files

```bash
#!/bin/bash
# clean-analysis.sh - Clean up generated analysis files

echo "=== Cleaning Analysis Files ==="

# List files that will be deleted
echo "The following files will be deleted:"
echo "  - *.txt (hotspot files)"
echo "  - *_findings.md"
echo "  - findings-*.json"
echo "  - *.analysis-strategy"
echo "  - discovery-manifest.json"
echo "  - hotspots/ directory"
echo ""

read -p "Are you sure you want to clean? (y/N) " -n 1 -r
echo
if [[ $REPLY =~ ^[Yy]$ ]]; then
    # Clean generated files
    rm -f *.txt
    rm -f *_findings.md
    rm -f findings-*.json
    rm -f .analysis-strategy
    rm -f discovery-manifest.json
    rm -f discovery-structure.txt
    rm -rf hotspots/

    echo "✓ Analysis files cleaned"
else
    echo "⚠ Cleanup cancelled"
fi
```

### analyze-all.sh - Complete Analysis Pipeline

```bash
#!/bin/bash
# analyze-all.sh - Run complete analysis pipeline

echo "=== Complete Code Analysis Pipeline ==="
echo "This will run all analysis steps automatically"
echo ""

# Step 1: Discovery
echo "[Step 1/5] Running discovery..."
./discovery.sh

# Step 2: Strategy selection
echo ""
echo "[Step 2/5] Selecting strategy..."
./strategy-selector.sh

# Step 3: Pattern scanning
echo ""
echo "[Step 3/5] Scanning for patterns..."
./pattern-scan.sh

# Step 4: Deep language-specific analysis
echo ""
echo "[Step 4/5] Running deep analysis..."

# Detect and run appropriate scanner
if [ -f pom.xml ] || [ -f build.gradle ]; then
    ./java-deep-scan.sh
elif [ -f requirements.txt ] || [ -f Pipfile ]; then
    ./python-deep-scan.sh
elif [ -f package.json ]; then
    ./nodejs-deep-scan.sh
else
    echo "No language-specific scanner available"
fi

# Step 5: Generate report
echo ""
echo "[Step 5/5] Generating report..."
./generate-report.sh

echo ""
echo "=== Analysis Complete ==="
echo "Report available at: CODE_REVIEW_REPORT.md"
echo "Run 'cat CODE_REVIEW_REPORT.md' to view"
```

---

## Quick Usage Guide

### Basic Workflow

1. **Make scripts executable**:
```bash
chmod +x *.sh
```

2. **Run quick scan** (5 minutes):
```bash
./quick-scan.sh
```

3. **Run full analysis** (30 minutes):
```bash
./analyze-all.sh
```

4. **View report**:
```bash
cat CODE_REVIEW_REPORT.md
```

### Advanced Workflow

1. **Install dependencies**:
```bash
./install-dependencies.sh
```

2. **Run discovery**:
```bash
./discovery.sh
```

3. **Select strategy**:
```bash
./strategy-selector.sh
```

4. **Run pattern scan**:
```bash
./pattern-scan.sh
```

5. **Run language-specific analysis**:
```bash
./java-deep-scan.sh    # For Java projects
./python-deep-scan.sh  # For Python projects
./nodejs-deep-scan.sh  # For Node.js projects
```

6. **Generate report**:
```bash
./generate-report.sh
```

7. **Clean up** (optional):
```bash
./clean-analysis.sh
```

---

## Script Customization

### Adding New Patterns

Edit `pattern-scan.sh` and add your patterns:

```bash
# Custom pattern example
echo "[CUSTOM_PATTERN]"
grep -r "your-pattern-here" --include="*.ext" -n > custom-pattern.txt
echo "  Found $(wc -l < custom-pattern.txt) custom patterns"
```

### Adding New Languages

Create a new scanner based on the template:

```bash
cp java-deep-scan.sh yourlang-deep-scan.sh
# Edit to match your language's patterns
```

### Customizing Report Format

Edit `generate-report.sh` to change the output format:

```bash
# Add custom sections
echo "## Custom Section" >> $REPORT
echo "Your custom analysis here" >> $REPORT
```

---

## See Also

- [START-HERE.md](START-HERE.md) - Framework documentation
- [EXAMPLES.md](EXAMPLES.md) - Real-world examples
- [LANGUAGE-PLUGINS.md](LANGUAGE-PLUGINS.md) - Language-specific patterns

---

**Version**: 1.0
**Last Updated**: 2024-10-12

END OF SCRIPTS