# QUICK START GUIDE

Get started with the Universal Code Analysis Framework in 30 minutes.

---

## PREREQUISITES

- Claude Code CLI installed and configured
- Access to a code repository to analyze
- Basic command-line familiarity
- `jq` for JSON processing (optional but recommended)

---

## 5-MINUTE QUICK START

### Step 1: Clone Your Target Repository

```bash
cd ~/projects
git clone https://github.com/your-org/your-repo.git
cd your-repo
```

### Step 2: Run Discovery

```bash
# Quick repository scan
echo "=== Repository Statistics ==="
find . -name "*.java" | wc -l
find . -name "*.py" | wc -l
find . -name "*.js" | wc -l

echo "=== Framework Detection ==="
test -f pom.xml && echo "Maven/Java"
test -f requirements.txt && echo "Python"
test -f package.json && echo "Node.js"

echo "=== Total LOC ==="
find . -name "*.java" -o -name "*.py" -o -name "*.js" | xargs wc -l | tail -1
```

### Step 3: Quick Security Scan

```bash
# Find critical security issues (5 seconds)
echo "=== Security Hotspots ==="

# SQL Injection
grep -r "query.*+" --include="*.java" --include="*.py" --include="*.js" -n | head -10

# Hardcoded secrets
grep -r "password.*=.*\"" --include="*.java" --include="*.properties" --include="*.py" -n | head -10

# Missing authentication
grep -r "@GetMapping\|@PostMapping" --include="*Controller.java" -A 2 | grep -v "@PreAuthorize" | head -10
```

**Result**: You now have hotspots to investigate!

---

## 30-MINUTE FULL ANALYSIS

### Complete Workflow

#### Phase 1: Discovery (5 minutes)

Create a discovery script:

```bash
#!/bin/bash
# File: discover.sh

OUTPUT="discovery-output.txt"

echo "=== CODE ANALYSIS DISCOVERY ===" > $OUTPUT
echo "Date: $(date)" >> $OUTPUT
echo "" >> $OUTPUT

echo "=== Repository Info ===" >> $OUTPUT
echo "Path: $(pwd)" >> $OUTPUT
echo "Git Remote: $(git remote get-url origin 2>/dev/null || echo 'N/A')" >> $OUTPUT
echo "" >> $OUTPUT

echo "=== Language Detection ===" >> $OUTPUT
echo "Java files: $(find . -name '*.java' 2>/dev/null | wc -l)" >> $OUTPUT
echo "Python files: $(find . -name '*.py' 2>/dev/null | wc -l)" >> $OUTPUT
echo "JavaScript files: $(find . -name '*.js' 2>/dev/null | wc -l)" >> $OUTPUT
echo "" >> $OUTPUT

echo "=== Framework Detection ===" >> $OUTPUT
test -f pom.xml && echo "Maven detected" >> $OUTPUT
test -f build.gradle && echo "Gradle detected" >> $OUTPUT
test -f requirements.txt && echo "Python pip detected" >> $OUTPUT
test -f package.json && echo "Node.js detected" >> $OUTPUT
echo "" >> $OUTPUT

echo "=== Project Structure ===" >> $OUTPUT
tree -L 3 -I 'node_modules|target|build|dist|__pycache__' >> $OUTPUT 2>/dev/null || \
  find . -maxdepth 3 -type d | head -50 >> $OUTPUT

echo "" >> $OUTPUT
echo "=== Total Lines of Code ===" >> $OUTPUT
find . \( -name "*.java" -o -name "*.py" -o -name "*.js" \) -exec wc -l {} + | \
  tail -1 >> $OUTPUT

cat $OUTPUT
```

Run it:
```bash
chmod +x discover.sh
./discover.sh
```

#### Phase 2: Pattern Scanning (10 minutes)

Create a pattern scan script:

```bash
#!/bin/bash
# File: pattern-scan.sh

OUTPUT="pattern-scan.json"

echo "{" > $OUTPUT

# Security patterns
echo '  "security": [' >> $OUTPUT

grep -r "password.*=.*\"" --include="*.java" --include="*.properties" --include="*.py" -n 2>/dev/null | \
  awk -F: '{print "    {\"file\": \"" $1 "\", \"line\": " $2 ", \"pattern\": \"hardcoded_password\"},"}' >> $OUTPUT

grep -r "query.*+" --include="*.java" --include="*.py" -n 2>/dev/null | \
  awk -F: '{print "    {\"file\": \"" $1 "\", \"line\": " $2 ", \"pattern\": \"sql_injection\"},"}' >> $OUTPUT

echo '  ],' >> $OUTPUT

# Performance patterns
echo '  "performance": [' >> $OUTPUT

grep -r "\.saveAll(" --include="*.java" -n 2>/dev/null | \
  awk -F: '{print "    {\"file\": \"" $1 "\", \"line\": " $2 ", \"pattern\": \"batch_operation\"},"}' >> $OUTPUT

grep -r "for.*for.*for" --include="*.java" --include="*.py" --include="*.js" -n 2>/dev/null | \
  awk -F: '{print "    {\"file\": \"" $1 "\", \"line\": " $2 ", \"pattern\": \"nested_loops\"},"}' >> $OUTPUT

echo '  ],' >> $OUTPUT

# Concurrency patterns
echo '  "concurrency": [' >> $OUTPUT

grep -r "parallelStream()" --include="*.java" -n 2>/dev/null | \
  awk -F: '{print "    {\"file\": \"" $1 "\", \"line\": " $2 ", \"pattern\": \"parallel_stream\"},"}' >> $OUTPUT

grep -r "threading\." --include="*.py" -n 2>/dev/null | \
  awk -F: '{print "    {\"file\": \"" $1 "\", \"line\": " $2 ", \"pattern\": \"threading\"},"}' >> $OUTPUT

echo '  ]' >> $OUTPUT
echo "}" >> $OUTPUT

echo "Pattern scan complete. Results in $OUTPUT"
cat $OUTPUT | jq . 2>/dev/null || cat $OUTPUT
```

Run it:
```bash
chmod +x pattern-scan.sh
./pattern-scan.sh
```

#### Phase 3: Deep Analysis with Agents (15 minutes)

Now use Claude Code agents for deep analysis:

**Security Agent**:

```markdown
# Save as: prompts/security-agent-prompt.md

# SECURITY ANALYSIS TASK

## Mission
Perform comprehensive security analysis of the codebase.

## Context
Project: [Your project name]
Language: [Detected from discovery]
Framework: [Detected from discovery]

## Hotspots (from pattern scan)
[Paste results from pattern-scan.json security section]

## Analysis Scope
Analyze the following for security vulnerabilities:
1. Input validation gaps
2. SQL injection vectors
3. Authentication/authorization bypass
4. Hardcoded secrets
5. Weak cryptography

## Output Format
Return JSON array of findings with this structure:
```json
[
  {
    "id": "SEC-CRIT-001",
    "type": "SECURITY",
    "severity": "CRITICAL",
    "category": "SQL_INJECTION",
    "file": "path/to/file",
    "line": 123,
    "evidence": "code snippet",
    "description": "what was found",
    "impact": "concrete impact",
    "recommendation": "actionable fix"
  }
]
```

## Instructions
1. Read the hotspot files identified above
2. Analyze each file thoroughly
3. Document every security issue found
4. Provide code evidence for each finding
5. Include fix recommendations

Start analysis now.
```

Launch the agent:

```bash
# Using Claude Code CLI
claude-code run-agent \
  --prompt-file prompts/security-agent-prompt.md \
  --output findings-security.json
```

**Performance Agent**:

```markdown
# Save as: prompts/performance-agent-prompt.md

# PERFORMANCE ANALYSIS TASK

## Mission
Identify performance bottlenecks in database queries, algorithms, and resource usage.

## Context
[Same as security agent]

## Hotspots
[Paste results from pattern-scan.json performance section]

## Analysis Scope
1. N+1 query detection
2. Missing batch configurations
3. Algorithm complexity (nested loops)
4. Missing caching
5. Large transactions

## Output Format
[Same JSON structure as security agent, but type: "PERFORMANCE"]

Start analysis now.
```

Launch:
```bash
claude-code run-agent \
  --prompt-file prompts/performance-agent-prompt.md \
  --output findings-performance.json
```

**Concurrency Agent**:

```markdown
# Save as: prompts/concurrency-agent-prompt.md

# CONCURRENCY ANALYSIS TASK

## Mission
Identify thread safety issues, race conditions, and resource leaks.

## Context
[Same as above]

## Hotspots
[Paste results from pattern-scan.json concurrency section]

## Analysis Scope
1. Non-thread-safe collections in parallel operations
2. ExecutorService lifecycle issues
3. Shared mutable state
4. Missing synchronization
5. Deadlock risks

## Output Format
[Same JSON structure, type: "CONCURRENCY"]

Start analysis now.
```

Launch:
```bash
claude-code run-agent \
  --prompt-file prompts/concurrency-agent-prompt.md \
  --output findings-concurrency.json
```

#### Phase 4: Merge Results

```bash
#!/bin/bash
# File: merge-findings.sh

echo "[" > findings-all.json

# Merge all findings
cat findings-security.json findings-performance.json findings-concurrency.json | \
  jq -s 'add' >> findings-all.json

echo "]" >> findings-all.json

echo "Merged findings into findings-all.json"

# Show summary
echo ""
echo "=== FINDINGS SUMMARY ==="
echo "Total findings: $(cat findings-all.json | jq 'length')"
echo "Critical: $(cat findings-all.json | jq '[.[] | select(.severity=="CRITICAL")] | length')"
echo "High: $(cat findings-all.json | jq '[.[] | select(.severity=="HIGH")] | length')"
echo "Medium: $(cat findings-all.json | jq '[.[] | select(.severity=="MEDIUM")] | length')"
echo "Low: $(cat findings-all.json | jq '[.[] | select(.severity=="LOW")] | length')"
```

Run:
```bash
chmod +x merge-findings.sh
./merge-findings.sh
```

---

## REAL-WORLD EXAMPLE: Spring Boot Application

### Repository Structure

```
spring-boot-app/
├── src/main/java/com/example/
│   ├── controller/      # REST controllers
│   ├── service/         # Business logic
│   ├── repository/      # Data access
│   ├── model/           # JPA entities
│   └── config/          # Configuration
├── src/main/resources/
│   └── application.yml
└── pom.xml
```

### Analysis Workflow

#### 1. Quick Discovery

```bash
cd spring-boot-app

# Framework detection
grep "<artifactId>spring-boot-starter" pom.xml
# Output: spring-boot-starter-web, spring-boot-starter-data-jpa

# Count entities
find . -name "*.java" -exec grep -l "@Entity" {} \; | wc -l
# Output: 45 entities

# Count repositories
find . -name "*Repository.java" | wc -l
# Output: 45 repositories

# Count controllers
find . -name "*Controller.java" | wc -l
# Output: 12 controllers
```

#### 2. Security Scan

```bash
# Find unprotected endpoints
grep -r "@GetMapping\|@PostMapping\|@DeleteMapping" src/main/java --include="*Controller.java" -A 3 | \
  grep -B 3 "public " | \
  grep -v "@PreAuthorize" | \
  head -20
```

**Result**:
```
UserController.java:45:    @DeleteMapping("/users/{id}")
UserController.java-46-    public ResponseEntity<?> deleteUser(@PathVariable Long id) {
UserController.java-47-        userService.deleteUser(id);
--
AdminController.java:23:    @PostMapping("/admin/reset-passwords")
AdminController.java-24-    public ResponseEntity<?> resetAllPasswords() {
AdminController.java-25-        adminService.resetPasswords();
```

**Finding**: 2 CRITICAL issues - unprotected admin endpoints!

#### 3. Performance Scan

```bash
# Check Hibernate batch config
grep -r "hibernate.jdbc.batch_size" src/main/resources
# Output: (empty) = MISSING!

# Find saveAll operations
grep -r "\.saveAll(" src/main/java --include="*.java" -n
```

**Result**:
```
EventService.java:234:        eventRepository.saveAll(events);
UserService.java:456:        userRepository.saveAll(users);
LicenseService.java:123:        licenseRepository.saveAll(licenses);
```

**Finding**: 3 locations using saveAll() without batch configuration = CRITICAL performance issue!

#### 4. Check for N+1 Queries

```bash
# Find lazy relationships
grep -r "@OneToMany\|@ManyToOne" src/main/java --include="*.java" -A 1 | \
  grep -B 1 "fetch.*LAZY" | \
  grep -v "@BatchSize"
```

**Result**:
```
User.java:45:    @OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
User.java-46-    private List<Order> orders;
--
Order.java:23:    @OneToMany(mappedBy = "order", fetch = FetchType.LAZY)
Order.java-24-    private List<OrderItem> items;
```

**Finding**: 15+ lazy relationships without @BatchSize = HIGH priority!

#### 5. Generate Report

```bash
# Create report from findings
cat > CODE_REVIEW_REPORT.md <<EOF
# Code Review Report - Spring Boot Application

## Executive Summary
- **Total Findings**: 45
- **Critical**: 5
- **High**: 12
- **Medium**: 18
- **Low**: 10

## Critical Issues

### [SEC-CRIT-001] Unprotected Admin Endpoint
**File**: \`UserController.java:45\`
**Severity**: CRITICAL

**Code**:
\`\`\`java
@DeleteMapping("/users/{id}")
public ResponseEntity<?> deleteUser(@PathVariable Long id) {
    // NO @PreAuthorize!
    userService.deleteUser(id);
    return ResponseEntity.ok().build();
}
\`\`\`

**Impact**: Any unauthenticated user can delete user accounts.

**Fix**:
\`\`\`java
@PreAuthorize("hasRole('ADMIN')")
@DeleteMapping("/users/{id}")
public ResponseEntity<?> deleteUser(@PathVariable Long id) {
    userService.deleteUser(id);
    return ResponseEntity.ok().build();
}
\`\`\`

### [PERF-CRIT-001] Missing Hibernate Batch Configuration
**File**: \`application.yml\`
**Severity**: CRITICAL

**Problem**: No hibernate.jdbc.batch_size configured. Found 3 usages of saveAll() that execute N individual INSERTs.

**Impact**: 80-90% slower bulk inserts.

**Fix**:
\`\`\`yaml
spring:
  jpa:
    properties:
      hibernate:
        jdbc.batch_size: 25
        order_inserts: true
        order_updates: true
\`\`\`

[Continue with remaining findings...]

## Quick Wins (1-Hour Fixes)

1. Add batch configuration (above) - 10 minutes
2. Add @PreAuthorize to admin endpoints - 20 minutes
3. Add @BatchSize(size=10) to 15 lazy relationships - 30 minutes

**Total time**: 1 hour
**Performance improvement**: 3-5x faster
**Security improvement**: Close 2 critical vulnerabilities
EOF

cat CODE_REVIEW_REPORT.md
```

---

## EXAMPLE OUTPUT

### Sample Finding (JSON)

```json
{
  "id": "SEC-CRIT-001",
  "type": "SECURITY",
  "severity": "CRITICAL",
  "category": "MISSING_AUTHENTICATION",
  "file": "src/main/java/com/example/controller/AdminController.java",
  "line": 45,
  "evidence": "@DeleteMapping(\"/users/{id}\")\npublic ResponseEntity<?> deleteUser(@PathVariable Long id) {\n    userService.deleteUser(id);\n    return ResponseEntity.ok().build();\n}",
  "description": "Admin endpoint for user deletion has no @PreAuthorize or @Secured annotation",
  "impact": "Any unauthenticated user can delete any user account by calling this endpoint. Tested via: curl -X DELETE http://localhost:8080/users/123",
  "recommendation": "Add authorization:\n@PreAuthorize(\"hasRole('ADMIN')\")\n@DeleteMapping(\"/users/{id}\")\npublic ResponseEntity<?> deleteUser(@PathVariable Long id) { ... }"
}
```

### Sample Report (Markdown)

```markdown
# Code Review Report

## Statistics
- **Total Findings**: 156
- **CRITICAL**: 8 issues
- **HIGH**: 34 issues
- **MEDIUM**: 78 issues
- **LOW**: 36 issues

## Top Priority Issues

### 1. SQL Injection in UserService
**Severity**: CRITICAL
**File**: `UserService.java:234`

Attack vector discovered allowing arbitrary SQL execution...

### 2. Missing Hibernate Batch Configuration
**Severity**: CRITICAL
**File**: `application.yml`

Bulk operations running 80% slower than optimal...

### 3. ExecutorService Thread Leak
**Severity**: CRITICAL
**File**: `NotificationService.java:89`

Thread pool leak causes OutOfMemoryError after 100 calls...

[Full report continues...]
```

---

## TROUBLESHOOTING

### Issue: "Too many files to analyze"

**Solution**: Use semantic segmentation

```bash
# Split by layer
find src/main/java -name "*Controller.java" > controllers.txt
find src/main/java -name "*Service*.java" > services.txt
find src/main/java -name "*Repository.java" > repositories.txt

# Analyze each layer with separate agent
claude-code run-agent --files controllers.txt --prompt security-prompt.md
claude-code run-agent --files services.txt --prompt performance-prompt.md
```

### Issue: "Agent exceeds token budget"

**Solution**: Use pattern scanning first

```bash
# Only analyze files with hotspots
grep -r "parallelStream()" --include="*.java" -l > hotspots.txt

# Pass only hotspot files to agent
claude-code run-agent --files hotspots.txt --prompt concurrency-prompt.md
```

### Issue: "Can't determine framework"

**Solution**: Manual specification

```bash
# Create manifest manually
cat > manifest.json <<EOF
{
  "project_name": "my-app",
  "languages": ["Java"],
  "frameworks": ["Spring Boot", "Hibernate"],
  "architecture": "microservice"
}
EOF

# Pass to agent
claude-code run-agent --context manifest.json --prompt security-prompt.md
```

---

## NEXT STEPS

1. **Automate**: Create CI/CD integration
2. **Customize**: Add project-specific patterns
3. **Track**: Monitor findings over time
4. **Fix**: Prioritize CRITICAL and HIGH issues
5. **Repeat**: Run analysis after major changes

---

## ADVANCED: Python Example

```bash
# Quick Python Django analysis
cd django-project

# Find views without authentication
grep -r "def.*request" --include="views.py" -B 2 | \
  grep -v "@login_required\|@permission_required"

# Find ORM N+1 patterns
grep -r "\.all()\|\.filter(" --include="*.py" -A 5 | \
  grep "for.*:" | \
  head -20

# Check for SQL injection
grep -r "cursor.execute.*+" --include="*.py" -n

# Missing database indexes
grep -r "class.*Model" --include="models.py" -A 20 | \
  grep "models\.\w*Field" | \
  grep -v "db_index=True"
```

---

## ADVANCED: JavaScript Example

```bash
# Quick Node.js/Express analysis
cd nodejs-app

# Find unprotected routes
grep -r "app.get\|app.post\|router.get\|router.post" --include="*.js" -B 2 | \
  grep -v "isAuthenticated\|requireAuth"

# Find XSS vulnerabilities
grep -r "innerHTML\|res.send.*req\." --include="*.js" -n

# Find missing timeouts
grep -r "axios.get\|axios.post\|fetch(" --include="*.js" | \
  grep -v "timeout"

# Check for event loop blocking
grep -r "Sync(" --include="*.js" -n
```

---

This quick start should get you analyzing codebases effectively. For deeper understanding, see `CLAUDE-ANALYSIS-FRAMEWORK.md` for the complete methodology.
