# LANGUAGE PLUGINS

Language-specific pattern catalogs and detection rules for the code analysis framework.

---

## PLUGIN ARCHITECTURE

Each language plugin provides:

1. **Discovery Patterns** - Detect language and frameworks
2. **Grep Patterns** - Find hotspots quickly
3. **Analysis Rules** - Language-specific checks
4. **Code Examples** - Anti-patterns and fixes
5. **Severity Mappings** - Context-specific severity levels

---

## JAVA PLUGIN

### Framework Detection

```bash
# Language detection
find . -name "*.java" | head -1  # Java present

# Build system
test -f pom.xml && echo "Maven"
test -f build.gradle && echo "Gradle"

# Framework detection
grep -q "spring-boot-starter" pom.xml && echo "Spring Boot"
grep -q "org.springframework" pom.xml && echo "Spring Framework"
grep -q "hibernate-core" pom.xml && echo "Hibernate"
grep -q "feign-core" pom.xml && echo "Feign"
grep -q "resilience4j" pom.xml && echo "Resilience4j"
```

### Security Patterns

#### SQL Injection
```bash
# CRITICAL patterns
grep -r "createNativeQuery.*+" --include="*.java" -n
grep -r "createQuery.*+" --include="*.java" -n
grep -r "String.*query.*=.*\"SELECT.*\".*+" --include="*.java" -n
grep -r "jdbcTemplate.query.*+" --include="*.java" -n

# Example violations
grep -r "Statement.*execute" --include="*.java" -n  # Use PreparedStatement
grep -r "String.format.*SELECT" --include="*.java" -n
```

**Anti-Pattern**:
```java
// CRITICAL: SQL Injection via concatenation
String query = "SELECT * FROM users WHERE email = '" + userEmail + "'";
entityManager.createNativeQuery(query).getResultList();

// CRITICAL: SQL Injection via String.format
String query = String.format("SELECT * FROM users WHERE id = %s", userId);
jdbcTemplate.query(query, new UserMapper());
```

**Fix**:
```java
// Use parameterized query
String query = "SELECT * FROM users WHERE email = ?1";
entityManager.createNativeQuery(query, User.class)
    .setParameter(1, userEmail)
    .getResultList();

// Use PreparedStatement
String query = "SELECT * FROM users WHERE id = ?";
jdbcTemplate.query(query, new UserMapper(), userId);
```

#### Authentication & Authorization
```bash
# Find unprotected endpoints
grep -r "@GetMapping\|@PostMapping\|@PutMapping\|@DeleteMapping" --include="*Controller.java" -A 3 | \
  grep -v "@PreAuthorize\|@Secured\|@RolesAllowed"

# Hardcoded passwords/secrets
grep -r "password.*=.*\"" --include="*.java" --include="*.properties" --include="*.yml" -n
grep -r "secret.*=.*\"" --include="*.java" --include="*.properties" --include="*.yml" -n
grep -r "apikey.*=.*\"" --include="*.java" --include="*.properties" --include="*.yml" -n
```

**Anti-Pattern**:
```java
// CRITICAL: No authorization check
@RestController
@RequestMapping("/admin")
public class AdminController {
    @DeleteMapping("/users/{id}")
    public ResponseEntity<?> deleteUser(@PathVariable Long id) {
        // NO @PreAuthorize!
        userService.deleteUser(id);
        return ResponseEntity.ok().build();
    }
}

// HIGH: Hardcoded credentials
public class EmailService {
    private static final String SMTP_PASSWORD = "myPassword123";  // HARDCODED!
}
```

**Fix**:
```java
// Add authorization
@RestController
@RequestMapping("/admin")
public class AdminController {
    @PreAuthorize("hasRole('ADMIN')")
    @DeleteMapping("/users/{id}")
    public ResponseEntity<?> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.ok().build();
    }
}

// Externalize secrets
@Value("${smtp.password}")
private String smtpPassword;
```

#### Insecure Deserialization
```bash
grep -r "ObjectInputStream" --include="*.java" -n
grep -r "readObject()" --include="*.java" -n
grep -r "XMLDecoder" --include="*.java" -n
```

**Anti-Pattern**:
```java
// CRITICAL: Deserializing untrusted data
ObjectInputStream ois = new ObjectInputStream(request.getInputStream());
Object obj = ois.readObject();  // UNSAFE!
```

#### Weak Cryptography
```bash
grep -r "MessageDigest.getInstance(\"MD5\"\|\"SHA1\")" --include="*.java" -n
grep -r "Cipher.getInstance(\"DES\"\|\"RC4\")" --include="*.java" -n
grep -r "new Random()" --include="*.java" -n  # Should use SecureRandom
```

**Anti-Pattern**:
```java
// MEDIUM: Weak hashing
MessageDigest md = MessageDigest.getInstance("MD5");  // WEAK!
byte[] hash = md.digest(password.getBytes());

// HIGH: Predictable random
Random random = new Random();  // NOT CRYPTOGRAPHICALLY SECURE!
String token = String.valueOf(random.nextInt());
```

**Fix**:
```java
// Strong hashing
BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
String hash = encoder.encode(password);

// Secure random
SecureRandom secureRandom = new SecureRandom();
byte[] token = new byte[32];
secureRandom.nextBytes(token);
```

### Performance Patterns

#### Hibernate N+1 Queries
```bash
# Find lazy relationships without @BatchSize
grep -r "@OneToMany\|@ManyToOne\|@ManyToMany" --include="*.java" -A 2 | \
  grep -v "@BatchSize"

# Find potential N+1 loops
grep -r "for.*:.*\." --include="*.java" -A 5 | grep "get[A-Z].*().size()\|get[A-Z].*().isEmpty()"

# Find saveAll() operations
grep -r "\.saveAll(" --include="*.java" -n

# Check batch configuration
grep -r "jdbc.batch_size" --include="*.yml" --include="*.properties"
```

**Anti-Pattern**:
```java
// CRITICAL: N+1 query
@Entity
public class User {
    @OneToMany(fetch = FetchType.LAZY)  // NO @BatchSize!
    private List<Order> orders;
}

// In service:
List<User> users = userRepository.findAll();  // 1 query
for (User user : users) {
    user.getOrders().size();  // N additional queries!
}

// CRITICAL: Batch operation without batch config
List<Event> events = ...; // 1000 events
eventRepository.saveAll(events);  // 1000 individual INSERTs without batch config!
```

**Fix**:
```java
// Add @BatchSize
@Entity
public class User {
    @OneToMany(fetch = FetchType.LAZY)
    @BatchSize(size = 10)  // Fetch in batches of 10
    private List<Order> orders;
}

// OR use JOIN FETCH
@Query("SELECT DISTINCT u FROM User u LEFT JOIN FETCH u.orders")
List<User> findAllWithOrders();

// Configure batch in application.yml
spring:
  jpa:
    properties:
      hibernate:
        jdbc.batch_size: 25
        order_inserts: true
        order_updates: true
```

#### Nested Loops
```bash
grep -r "for.*for.*for" --include="*.java" -n
grep -r "while.*while" --include="*.java" -n
```

**Anti-Pattern**:
```java
// HIGH: O(n²) algorithm
for (User user : users) {  // O(n)
    for (Order order : orders) {  // O(m)
        if (order.getUserId().equals(user.getId())) {
            // match
        }
    }
}
// n=1000, m=5000 = 5,000,000 iterations!
```

**Fix**:
```java
// O(n+m) using HashMap
Map<Long, User> userMap = users.stream()
    .collect(Collectors.toMap(User::getId, u -> u));

for (Order order : orders) {  // O(m)
    User user = userMap.get(order.getUserId());  // O(1) lookup
    // instant match
}
// n=1000, m=5000 = 6,000 operations
```

#### Transaction Boundaries
```bash
grep -r "@Transactional" --include="*.java" -A 10 | grep "for.*{"
```

**Anti-Pattern**:
```java
// HIGH: Transaction too large
@Transactional  // Holds DB connection for entire file processing!
public void processFile(File file) {
    List<String> lines = readAllLines(file);  // 100,000 lines
    for (String line : lines) {
        Entity entity = parse(line);
        repository.save(entity);  // 100,000 saves in one transaction!
    }
}
```

**Fix**:
```java
// Process in chunks
public void processFile(File file) {
    List<String> lines = readAllLines(file);
    Lists.partition(lines, 1000).forEach(chunk -> {
        processChunk(chunk);  // Transaction per 1000 records
    });
}

@Transactional
private void processChunk(List<String> lines) {
    List<Entity> entities = lines.stream()
        .map(this::parse)
        .collect(Collectors.toList());
    repository.saveAll(entities);  // Batch insert 1000 at a time
}
```

### Concurrency Patterns

#### Thread Safety Issues
```bash
# Parallel stream with non-thread-safe collection
grep -r "parallelStream()" --include="*.java" -B 5 -A 5 | grep "ArrayList\|HashMap\|HashSet"

# ExecutorService without shutdown
grep -r "Executors\.new" --include="*.java" -A 10 | grep -v "shutdown()"

# Synchronized issues
grep -r "synchronized" --include="*.java" -n
```

**Anti-Pattern**:
```java
// CRITICAL: Race condition
List<Result> results = new ArrayList<>();  // NOT THREAD-SAFE!
data.parallelStream().forEach(item -> {
    results.add(process(item));  // CONCURRENT MODIFICATION!
});

// CRITICAL: ExecutorService leak
public void sendEmails(List<User> users) {
    ExecutorService executor = Executors.newFixedThreadPool(10);
    for (User user : users) {
        executor.submit(() -> sendEmail(user));
    }
    // NO shutdown()! THREAD LEAK!
}

// HIGH: Non-atomic operation
if (!cache.containsKey(key)) {  // CHECK
    cache.put(key, expensiveComputation());  // THEN ACT - RACE CONDITION!
}
```

**Fix**:
```java
// Use collect() for parallel streams
List<Result> results = data.parallelStream()
    .map(item -> process(item))
    .collect(Collectors.toList());  // THREAD-SAFE!

// OR use thread-safe collection
List<Result> results = new CopyOnWriteArrayList<>();
data.parallelStream().forEach(item -> {
    results.add(process(item));
});

// Always shutdown ExecutorService
ExecutorService executor = Executors.newFixedThreadPool(10);
try {
    for (User user : users) {
        executor.submit(() -> sendEmail(user));
    }
} finally {
    executor.shutdown();
    executor.awaitTermination(60, TimeUnit.SECONDS);
}

// Use atomic operation
cache.computeIfAbsent(key, k -> expensiveComputation());
```

### Resilience Patterns

#### Timeouts
```bash
# Feign timeout config
grep -r "connectTimeout\|readTimeout" --include="*.yml" --include="*.properties" -n

# RestTemplate without timeout
grep -r "new RestTemplate()" --include="*.java" -n
```

**Anti-Pattern**:
```yaml
# CRITICAL: Excessive timeout
feign:
  client:
    config:
      default:
        readTimeout: 60000  # 60 seconds!
      crm:
        readTimeout: 300000  # 5 MINUTES!!!
```

**Fix**:
```yaml
# Reasonable timeouts
feign:
  client:
    config:
      default:
        connectTimeout: 5000  # 5s
        readTimeout: 10000    # 10s
      crm:  # Known slow service
        connectTimeout: 5000
        readTimeout: 30000    # 30s max
```

#### Circuit Breakers
```bash
# Find Feign clients without circuit breakers
grep -r "@FeignClient" --include="*.java" -A 5 | grep -v "@CircuitBreaker"

# Check Resilience4j config
grep -r "resilience4j" --include="*.yml" --include="*.properties"
```

**Anti-Pattern**:
```java
// CRITICAL: No circuit breaker
@FeignClient(name = "payment-service")
public interface PaymentClient {
    @PostMapping("/charge")
    PaymentResponse charge(ChargeRequest request);
    // NO @CircuitBreaker - failures cascade!
}
```

**Fix**:
```java
// Add circuit breaker
@Service
public class PaymentService {
    @Autowired
    private PaymentClient paymentClient;

    @CircuitBreaker(name = "paymentService", fallbackMethod = "chargeFallback")
    public PaymentResponse charge(ChargeRequest request) {
        return paymentClient.charge(request);
    }

    private PaymentResponse chargeFallback(ChargeRequest request, Exception e) {
        log.error("Payment service unavailable", e);
        return PaymentResponse.error("Service temporarily unavailable");
    }
}

// application.yml
resilience4j:
  circuitbreaker:
    instances:
      paymentService:
        slidingWindowSize: 100
        failureRateThreshold: 50
        waitDurationInOpenState: 30s
```

---

## PYTHON PLUGIN

### Framework Detection

```bash
# Language detection
find . -name "*.py" | head -1

# Framework detection
test -f requirements.txt && echo "pip"
test -f Pipfile && echo "pipenv"
test -f pyproject.toml && echo "poetry"

grep -q "Django" requirements.txt && echo "Django"
grep -q "Flask" requirements.txt && echo "Flask"
grep -q "FastAPI" requirements.txt && echo "FastAPI"
grep -q "sqlalchemy" requirements.txt && echo "SQLAlchemy"
```

### Security Patterns

#### SQL Injection
```bash
# CRITICAL patterns
grep -r "cursor.execute.*+" --include="*.py" -n
grep -r "cursor.execute.*%" --include="*.py" -n
grep -r "cursor.execute.*format" --include="*.py" -n
grep -r ".raw(" --include="*.py" -n  # Django raw queries
grep -r ".extra(" --include="*.py" -n  # Django extra()
```

**Anti-Pattern**:
```python
# CRITICAL: SQL Injection via concatenation
cursor.execute("SELECT * FROM users WHERE email = '" + email + "'")

# CRITICAL: SQL Injection via format
cursor.execute("SELECT * FROM users WHERE id = {}".format(user_id))

# CRITICAL: Django raw query with f-string
User.objects.raw(f"SELECT * FROM users WHERE email = '{email}'")
```

**Fix**:
```python
# Use parameterized queries
cursor.execute("SELECT * FROM users WHERE email = %s", (email,))

# Django ORM (safe by default)
User.objects.filter(email=email)

# Django raw with parameters
User.objects.raw("SELECT * FROM users WHERE email = %s", [email])
```

#### Authentication & Authorization
```bash
# Find unprotected views
grep -r "def.*request" --include="views.py" -B 2 | grep -v "@login_required\|@permission_required"

# Hardcoded secrets
grep -r "PASSWORD.*=.*['\"]" --include="*.py" --include="settings.py" -n
grep -r "SECRET_KEY.*=.*['\"]" --include="*.py" -n
```

**Anti-Pattern**:
```python
# CRITICAL: No authentication
@app.route('/admin/delete-user/<int:user_id>')
def delete_user(user_id):
    # NO @login_required!
    User.query.get(user_id).delete()
    return jsonify({'success': True})

# HIGH: Hardcoded secret
SECRET_KEY = 'my-secret-key-123'  # HARDCODED!

# HIGH: Weak password hashing
import hashlib
password_hash = hashlib.md5(password.encode()).hexdigest()  # WEAK!
```

**Fix**:
```python
# Add authentication
from flask_login import login_required

@app.route('/admin/delete-user/<int:user_id>')
@login_required
def delete_user(user_id):
    if not current_user.is_admin:
        abort(403)
    User.query.get(user_id).delete()
    return jsonify({'success': True})

# Externalize secrets
import os
SECRET_KEY = os.environ.get('SECRET_KEY')

# Strong password hashing
from werkzeug.security import generate_password_hash
password_hash = generate_password_hash(password)
```

#### Command Injection
```bash
grep -r "os.system(" --include="*.py" -n
grep -r "subprocess.call.*shell=True" --include="*.py" -n
grep -r "eval(" --include="*.py" -n
grep -r "exec(" --include="*.py" -n
```

**Anti-Pattern**:
```python
# CRITICAL: Command injection
os.system(f"rm -rf {user_input}")  # ARBITRARY COMMAND EXECUTION!

# CRITICAL: Shell injection
subprocess.call(f"ping {host}", shell=True)  # UNSAFE!

# CRITICAL: Code injection
eval(request.form['code'])  # ARBITRARY CODE EXECUTION!
```

**Fix**:
```python
# Use array form (no shell)
subprocess.call(['rm', '-rf', user_input])

# Use run() with list
subprocess.run(['ping', host], check=True)

# Don't use eval/exec with user input (or at all)
# Use ast.literal_eval for safe literal evaluation
import ast
data = ast.literal_eval(safe_string)
```

### Performance Patterns

#### Django ORM N+1 Queries
```bash
# Find potential N+1 patterns
grep -r "\.all()\|\.filter(" --include="*.py" -A 5 | grep "for.*:"

# Check for select_related/prefetch_related
grep -r "select_related\|prefetch_related" --include="*.py"
```

**Anti-Pattern**:
```python
# CRITICAL: N+1 query
users = User.objects.all()  # 1 query
for user in users:  # N queries follow!
    print(user.profile.avatar)  # Extra query per user!
    print(user.orders.count())  # Extra query per user!

# HIGH: Missing database index
class User(models.Model):
    email = models.EmailField()  # NO db_index=True!
    # But frequently queried: User.objects.filter(email=...)
```

**Fix**:
```python
# Use select_related for foreign keys
users = User.objects.select_related('profile').all()  # 1 query with JOIN
for user in users:
    print(user.profile.avatar)  # No extra query!

# Use prefetch_related for reverse relationships
users = User.objects.prefetch_related('orders').all()
for user in users:
    print(user.orders.count())  # No extra queries!

# Add database index
class User(models.Model):
    email = models.EmailField(db_index=True)  # INDEX ADDED!
```

#### Algorithm Complexity
```bash
grep -r "for.*:" --include="*.py" -A 3 | grep "for.*:"
```

**Anti-Pattern**:
```python
# HIGH: O(n²) algorithm
for user in users:  # O(n)
    for order in orders:  # O(m)
        if order.user_id == user.id:
            # match
```

**Fix**:
```python
# O(n+m) using dictionary
user_dict = {user.id: user for user in users}  # O(n)
for order in orders:  # O(m)
    user = user_dict.get(order.user_id)  # O(1) lookup
```

### Concurrency Patterns

#### Thread Safety
```bash
grep -r "threading\." --include="*.py" -n
grep -r "global " --include="*.py" -n
grep -r "ThreadPoolExecutor" --include="*.py" -n
```

**Anti-Pattern**:
```python
# CRITICAL: Shared mutable state
results = []  # Global shared list!

def worker(item):
    result = process(item)
    results.append(result)  # NOT THREAD-SAFE!

with ThreadPoolExecutor() as executor:
    executor.map(worker, items)

# HIGH: Missing thread join
import threading

threads = [threading.Thread(target=worker, args=(item,)) for item in items]
for t in threads:
    t.start()
# MISSING: for t in threads: t.join()
```

**Fix**:
```python
# Use thread-safe queue
from queue import Queue

results = Queue()  # Thread-safe!

def worker(item):
    result = process(item)
    results.put(result)

# OR use return values with futures
from concurrent.futures import ThreadPoolExecutor

with ThreadPoolExecutor() as executor:
    futures = [executor.submit(process, item) for item in items]
    results = [f.result() for f in futures]

# Always join threads
threads = [threading.Thread(target=worker, args=(item,)) for item in items]
for t in threads:
    t.start()
for t in threads:
    t.join()  # WAIT FOR COMPLETION!
```

### Resilience Patterns

#### Timeouts
```bash
grep -r "requests.get\|requests.post" --include="*.py" -n | grep -v "timeout="
```

**Anti-Pattern**:
```python
# CRITICAL: No timeout
response = requests.get(external_url)  # HANGS FOREVER IF SLOW!

# HIGH: Timeout too high
response = requests.get(url, timeout=300)  # 5 minutes!
```

**Fix**:
```python
# Set reasonable timeout (connect, read)
response = requests.get(url, timeout=(5, 30))  # 5s connect, 30s read

# Use retry with exponential backoff
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(stop=stop_after_attempt(3),
       wait=wait_exponential(multiplier=1, min=1, max=10))
def fetch_data(url):
    return requests.get(url, timeout=(5, 30))
```

---

## JAVASCRIPT/NODE.JS PLUGIN

### Framework Detection

```bash
# Language detection
test -f package.json && echo "Node.js"

# Framework detection
grep -q "express" package.json && echo "Express"
grep -q "react" package.json && echo "React"
grep -q "mongoose" package.json && echo "Mongoose"
grep -q "sequelize" package.json && echo "Sequelize"
```

### Security Patterns

#### SQL Injection
```bash
grep -r "query.*+" --include="*.js" -n
grep -r "query.*\${" --include="*.js" -n
grep -r ".raw(" --include="*.js" -n
```

**Anti-Pattern**:
```javascript
// CRITICAL: SQL Injection
const query = "SELECT * FROM users WHERE id = " + userId;
db.query(query);

// CRITICAL: SQL Injection with template literal
const query = `SELECT * FROM users WHERE email = '${email}'`;
db.query(query);
```

**Fix**:
```javascript
// Use parameterized queries
const query = "SELECT * FROM users WHERE id = ?";
db.query(query, [userId]);

// OR use ORM
User.findOne({ where: { email: email } });  // Sequelize - safe
```

#### XSS (Cross-Site Scripting)
```bash
grep -r "innerHTML\|dangerouslySetInnerHTML" --include="*.js" --include="*.jsx" -n
grep -r "res.send.*req\." --include="*.js" -n
grep -r "eval(" --include="*.js" -n
```

**Anti-Pattern**:
```javascript
// CRITICAL: XSS via innerHTML
element.innerHTML = userInput;  // UNSANITIZED!

// CRITICAL: XSS in Express response
app.get('/greet', (req, res) => {
    res.send("<h1>Hello " + req.query.name + "</h1>");  // NO ESCAPING!
});

// CRITICAL: Code injection
eval(req.body.code);  // ARBITRARY CODE EXECUTION!
```

**Fix**:
```javascript
// Use textContent or escape
element.textContent = userInput;  // Safe

// Use template engine with auto-escaping
app.get('/greet', (req, res) => {
    res.render('greet', { name: req.query.name });  // Template escapes
});

// Never use eval with user input
// Use JSON.parse for data
const data = JSON.parse(req.body.data);
```

#### Authentication
```bash
grep -r "app.get\|app.post\|router.get\|router.post" --include="*.js" -B 2 | \
  grep -v "isAuthenticated\|requireAuth\|checkAuth"
```

**Anti-Pattern**:
```javascript
// CRITICAL: No authentication
app.delete('/api/users/:id', (req, res) => {
    // NO AUTH CHECK!
    User.destroy({ where: { id: req.params.id } });
    res.json({ success: true });
});
```

**Fix**:
```javascript
// Add authentication middleware
const requireAuth = (req, res, next) => {
    if (!req.isAuthenticated()) {
        return res.status(401).json({ error: 'Unauthorized' });
    }
    next();
};

app.delete('/api/users/:id', requireAuth, (req, res) => {
    // Check authorization
    if (!req.user.isAdmin) {
        return res.status(403).json({ error: 'Forbidden' });
    }
    User.destroy({ where: { id: req.params.id } });
    res.json({ success: true });
});
```

### Performance Patterns

#### Mongoose N+1
```bash
grep -r "\.find()\|\.findOne()" --include="*.js" -A 5 | grep "\.populate("
```

**Anti-Pattern**:
```javascript
// CRITICAL: N+1 query
const users = await User.find();  // 1 query
for (const user of users) {
    await user.populate('orders');  // N queries!
    console.log(user.orders.length);
}

// HIGH: Missing lean()
const users = await User.find();  // Returns Mongoose documents (heavy!)
```

**Fix**:
```javascript
// Use populate in initial query
const users = await User.find().populate('orders');  // 1 or 2 queries

// Use lean() for read-only data
const users = await User.find().lean();  // Plain objects (10x faster!)
```

#### Event Loop Blocking
```bash
grep -r "Sync(" --include="*.js" -n  # Synchronous operations
grep -r "for.*let.*<.*length" --include="*.js" -A 3 | grep "await"
```

**Anti-Pattern**:
```javascript
// CRITICAL: Blocking event loop
app.get('/process', (req, res) => {
    const data = fs.readFileSync(largefile);  // BLOCKS!
    res.send(processData(data));
});

// HIGH: CPU-intensive work on event loop
app.get('/calculate', (req, res) => {
    let result = 0;
    for (let i = 0; i < 1000000000; i++) {  // BLOCKS EVENT LOOP!
        result += i;
    }
    res.json({ result });
});
```

**Fix**:
```javascript
// Use async operations
app.get('/process', async (req, res) => {
    const data = await fs.promises.readFile(largefile);  // NON-BLOCKING!
    res.send(processData(data));
});

// Use worker threads for CPU-intensive work
const { Worker } = require('worker_threads');

app.get('/calculate', (req, res) => {
    const worker = new Worker('./calculate-worker.js');
    worker.on('message', result => {
        res.json({ result });
    });
});
```

### Concurrency Patterns

#### Promise Handling
```bash
grep -r "\.then(" --include="*.js" -n | grep -v "\.catch("
grep -r "new Promise(" --include="*.js" -A 10 | grep -v "reject"
```

**Anti-Pattern**:
```javascript
// HIGH: Unhandled promise rejection
doAsyncWork().then(result => {
    // handle success
});
// MISSING .catch()!

// MEDIUM: Promise constructor anti-pattern
new Promise((resolve, reject) => {
    doAsyncWork().then(result => resolve(result));  // Unnecessary wrapping!
});
```

**Fix**:
```javascript
// Always handle errors
doAsyncWork()
    .then(result => {
        // handle success
    })
    .catch(error => {
        console.error('Error:', error);
    });

// OR use async/await with try/catch
try {
    const result = await doAsyncWork();
} catch (error) {
    console.error('Error:', error);
}

// Don't wrap promises unnecessarily
async function wrapper() {
    return doAsyncWork();  // Returns promise directly
}
```

### Resilience Patterns

#### Timeouts
```bash
grep -r "axios.get\|axios.post\|fetch(" --include="*.js" -n | grep -v "timeout"
```

**Anti-Pattern**:
```javascript
// CRITICAL: No timeout
axios.get(externalUrl);  // CAN HANG INDEFINITELY!

// HIGH: Timeout too long
axios.get(url, { timeout: 60000 });  // 60 seconds!
```

**Fix**:
```javascript
// Set reasonable timeout
const client = axios.create({
    timeout: 30000,  // 30s
    validateStatus: status => status < 500
});

// Add retry with exponential backoff
import axiosRetry from 'axios-retry';

axiosRetry(client, {
    retries: 3,
    retryDelay: axiosRetry.exponentialDelay
});
```

---

## PATTERN SUMMARY TABLE

| Category | Java | Python | JavaScript |
|----------|------|--------|------------|
| SQL Injection | String concatenation in queries | cursor.execute with + or % | Template literals in queries |
| Auth Missing | No @PreAuthorize/@Secured | No @login_required | No auth middleware |
| Hardcoded Secrets | password/secret in .java/.properties | PASSWORD/SECRET_KEY in .py | API keys in .js/config |
| N+1 Queries | Lazy loading without @BatchSize | No select_related/prefetch_related | No populate in initial query |
| Thread Safety | parallelStream() + ArrayList | threading + global list | N/A (single-threaded) |
| Event Loop Blocking | N/A | N/A | Sync operations, CPU-intensive work |
| No Timeout | RestTemplate without config | requests without timeout | axios/fetch without timeout |
| No Circuit Breaker | @FeignClient without @CircuitBreaker | requests without retry | axios without retry |

---

## ADDING NEW LANGUAGES

### Template for New Language Plugin

```markdown
## [LANGUAGE] PLUGIN

### Framework Detection
```bash
# Language detection patterns
# Build system detection
# Framework detection
```

### Security Patterns
- SQL Injection patterns
- Authentication/Authorization patterns
- Input validation patterns
- Cryptography patterns

### Performance Patterns
- Database query optimization
- Algorithm complexity
- Caching patterns

### Concurrency Patterns
- Thread safety issues
- Async/await patterns

### Resilience Patterns
- Timeout configurations
- Retry mechanisms
- Circuit breakers

### Example Anti-Patterns and Fixes
[Provide language-specific examples]
```

---

This completes the language plugin catalog. Use these patterns in conjunction with the agent prompts from `AGENT-PROMPTS.md`.
