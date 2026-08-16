# Fencing Token Pattern: Problem & Solution

## The Problem (Without Fencing)

```
┌─────────┐         ┌──────┐         ┌─────────┐
│  A (lock holder)  │         │   Redis    │         │   Database   │
│  - Acquire lock   │────────▶│  lock=42   │────────▶│              │
│  - Start writing  │         │            │         │              │
│  - GC PAUSE!     │         │            │         │              │
│  (blocked)       │         │            │         │              │
│                   │         │ TTL expires │         │              │
│                   │         │ lock=43     │         │              │
│                   │         │  (to B)     │         │              │
│                   │         │            │         │              │
│  - Wake up        │         │            │────────▶│ OVERWRITE!  │
│  - Keep writing   │         │            │         │   Data lost  │
└─────────┘         └──────┘         └─────────┘
```

**Timeline:**
1. Instance A acquires lock with token 42
2. Instance A starts writing to DB
3. GC pause happens (A is blocked)
4. TTL expires, Redis gives lock to instance C with token 43
5. Instance C writes with token 43
6. A wakes up, continues writing with token 42 → **OVERWRITES C's data!**

---

## The Solution (With Fencing Token)

```
┌─────────┐         ┌──────┐         ┌─────────┐
│  A (token=42)    │         │   Redis    │         │   Database   │
│  - Acquire lock  │────────▶│  lock=42   │────────▶│  max=42      │
│  - Write + 42   │         │  token=42  │────────▶│  ✓ accepted │
│  - GC PAUSE!    │         │            │         │              │
│                   │         │ TTL expires │         │              │
│                   │         │ lock=43     │         │              │
│                   │         │ token=43    │         │              │
│                   │         │            │────────▶│  token=43    │
│                   │         │            │         │  max=43 ✓   │
│                   │         │            │         │              │
│  - Wake up       │         │            │────────▶│  token=42    │
│  - Write + 42   │         │            │         │  max=43 ✗   │
│    (stale!)     │         │            │         │  REJECTED!   │
└─────────┘         └──────┘         └─────────┘
```

**Key insight:** Database tracks the highest token seen and rejects older ones.

---

## Implementation Example (Go)

### 1. Redis Lock with Fencing Token

```go
package fencing

import (
    "context"
    "errors"
    "strconv"
    "time"

    "github.com/redis/go-redis/v9"
)

var (
    ErrLockNotAcquired = errors.New("lock not acquired")
)

type FencedLock struct {
    client     *redis.Client
    key        string
    token      int64  // The fencing token
    ttl        time.Duration
}

func (f *FencedLock) Token() int64 {
    return f.token
}

// AcquireLockWithFencing acquires a lock and returns a fencing token
func AcquireLockWithFencing(ctx context.Context, client *redis.Client, key string, ttl time.Duration) (*FencedLock, error) {
    // Use INCR to get monotonic token
    tokenKey := key + ":token"
    
    // Atomically increment token and acquire lock
    pipe := client.Pipeline()
    
    // Increment token counter
    incr := pipe.Incr(ctx, tokenKey)
    // Set lock with token as value
    set := pipe.SetNX(ctx, key, incr.Val(), ttl)
    
    _, err := pipe.Exec(ctx)
    if err != nil {
        return nil, err
    }
    
    if !set.Val() {
        return nil, ErrLockNotAcquired
    }
    
    return &FencedLock{
        client: client,
        key:    key,
        token:  incr.Val(),
        ttl:    ttl,
    }, nil
}

func (f *FencedLock) Release(ctx context.Context) error {
    // Use Lua script to ensure atomic delete only if we still hold the lock
    script := redis.NewScript(`
        if redis.call("get", KEYS[1]) == ARGV[1] then
            return redis.call("del", KEYS[1])
        else
            return 0
        end
    `)
    
    _, err := script.Run(ctx, f.client, []string{f.key}, f.token).Result()
    return err
}

func (f *FencedLock) Extend(ctx context.Context, ttl time.Duration) error {
    script := redis.NewScript(`
        if redis.call("get", KEYS[1]) == ARGV[1] then
            return redis.call("pexpire", KEYS[1], ARGV[2])
        else
            return 0
        end
    `)
    
    result, err := script.Run(ctx, f.client, []string{f.key}, f.token, ttl.Milliseconds()).Result()
    if result.(int64) == 0 {
        return ErrLockNotAcquired
    }
    return err
}
```

### 2. Database with Fencing Check

```go
package fencing

import (
    "context"
    "database/sql"
    "errors"
)

var (
    ErrStaleToken = errors.New("stale fencing token")
)

type Account struct {
    ID           int64
    Balance      int64
    FencingToken int64  // Track highest token
}

type AccountRepository struct {
    db *sql.DB
}

// WriteWithFencing writes only if token is newer than recorded
func (r *AccountRepository) WriteWithFencing(ctx context.Context, id int64, amount int64, token int64) error {
    // Use conditional update: only succeed if token > current max_token
    query := `
        UPDATE accounts 
        SET balance = balance + ?,
            fencing_token = ?
        WHERE id = ? AND fencing_token < ?
        RETURNING fencing_token
    `
    
    var newToken int64
    err := r.db.QueryRowContext(ctx, query, amount, token, id, token).Scan(&newToken)
    if err == sql.ErrNoRows {
        return ErrStaleToken
    }
    if err != nil {
        return err
    }
    
    return nil
}

// ReadAccount reads current account state
func (r *AccountRepository) ReadAccount(ctx context.Context, id int64) (*Account, error) {
    var acc Account
    err := r.db.QueryRowContext(ctx, 
        "SELECT id, balance, fencing_token FROM accounts WHERE id = ?", id).Scan(
        &acc.ID, &acc.Balance, &acc.FencingToken,
    )
    if err != nil {
        return nil, err
    }
    return &acc, nil
}
```

### 3. Usage in Service

```go
func ProcessPayment(ctx context.Context, client *redis.Client, repo *AccountRepository, accountID int64, amount int64) error {
    // 1. Acquire lock with fencing token
    lock, err := AcquireLockWithFencing(ctx, client, "account:"+strconv.FormatInt(accountID, 10), 10*time.Second)
    if err != nil {
        return err
    }
    defer lock.Release(ctx)
    
    // 2. Do work (with token)
    err = repo.WriteWithFencing(ctx, accountID, amount, lock.Token())
    if errors.Is(err, ErrStaleToken) {
        // Another instance wrote with higher token - our write was rejected
        return errors.New("concurrent modification detected, please retry")
    }
    return err
}
```

---

## Best Practices

### 1. Always Use Fencing Tokens
- Lock alone is not sufficient for distributed systems
- The resource (DB) must enforce the token check

### 2. Token Must Be Monotonic
- Use Redis INCR or database sequence
- Never reuse or guess tokens

### 3. Short TTL with Extension
- Keep TTL short (5-30 seconds)
- Use lock extension for long-running operations
- Extension must include token verification

### 4. Idempotency is Key
- Design operations to be idempotent
- When fence rejection happens, either:
  - Retry with fresh lock
  - Use idempotency keys

### 5. Separate Lock & Token Storage
```go
// Good: separate keys
lock:key     = owner_id        (TTL: 10s)
lock:token   = 43              (no TTL, keeps incrementing)

// Alternative: store together in Lua
// But keep token counter persistent
```

### 6. Handle Edge Cases
- Lock holder dies: token is lost → new holder gets higher token → safe
- Network partition: token check protects against split-brain
- Clock skew: doesn't matter (token is logical, not time-based)

---

## Real-World Examples

### DynamoDB
- Use conditional writes with `ExpectedVersion`
- Or use DynamoDB Lock Client (already implements fencing)

### PostgreSQL
- Use `SELECT ... FOR UPDATE` with additional token column
- Or use advisory locks with external token tracking

### etcd/Consul
- Many distributed systems have built-in lease + token concepts

### Kafka
- Idempotent producers use monotonic sequence numbers
- Exactly-once semantics include fencing

---

## Real Scenario: Job Scheduler with Redis Lock

### The Problem

```
Job: generate-daily-report (runs every 5 minutes)
3 instances behind ALB

Timeline:
─────────────────────────────────────────────────────────────
T+0:    Instance B acquires lock (job:daily-report, TTL=300s)
T+60:   B's pod OOM-killed → job NOT done, lock still valid
T+240:  Instance A grabs lock → tries to run job
T+240:  Instance B's pod restarts → ALSO tries to run job
        → BOTH running same job! DUPLICATE EXECUTION!

Or worse (GC pause):
T+0:    A acquires lock (token=42), starts job
T+290:  A has GC pause, still holding lock
T+300:  TTL expires, Redis gives lock to C (token=43)
T+310:  A wakes up, continues with token=42 (thinks it owns lock)
T+310:  C starts with token=43
        → BOTH think they own lock!
```

### The Solution: Lock + Token + Job State

```go
package scheduler

import (
    "context"
    "errors"
    "time"

    "github.com/redis/go-redis/v9"
)

var (
    ErrJobAlreadyRunning = errors.New("job already running")
    ErrStaleToken        = errors.New("stale fencing token")
    ErrJobNotFound       = errors.New("job not found")
)

type JobStatus string

const (
    JobStatusPending   JobStatus = "pending"
    JobStatusRunning   JobStatus = "running"
    JobStatusCompleted JobStatus = "completed"
    JobStatusFailed    JobStatus = "failed"
)

type Job struct {
    Name          string
    Status        JobStatus
    FencingToken  int64
    StartedAt     time.Time
    InstanceID    string
}

type JobScheduler struct {
    redis     *redis.Client
    instanceID string
}

// AcquireJobLock acquires lock with fencing token AND checks job state
func (s *JobScheduler) AcquireJobLock(ctx context.Context, jobName string, ttl time.Duration) (int64, error) {
    lockKey := "job:lock:" + jobName
    tokenKey := "job:token:" + jobName
    stateKey := "job:state:" + jobName

    // Use Lua script for atomic operation
    script := redis.NewScript(`
        -- Increment token to get new fencing token
        local token = redis.call('INCR', KEYS[2])
        
        -- Try to acquire lock
        local acquired = redis.call('SET', KEYS[1], ARGV[1], 'NX', 'EX', ARGV[2])
        
        if not acquired then
            -- Lock already held, check if same instance
            local currentHolder = redis.call('GET', KEYS[1])
            if currentHolder == ARGV[1] then
                -- Same instance, extend lock and use same token
                redis.call('EXPIRE', KEYS[1], ARGV[2])
                token = tonumber(redis.call('GET', KEYS[2]))
            else
                -- Different instance holds lock
                return -1
            end
        end
        
        -- Initialize job state if not exists
        if not redis.call('EXISTS', KEYS[3]) then
            redis.call('HMSET', KEYS[3], 
                'status', 'pending', 
                'fencing_token', '0',
                'instance_id', '')
        end
        
        return token
    `)

    result, err := script.Run(ctx, s.redis,
        []string{lockKey, tokenKey, stateKey},
        s.instanceID, int(ttl.Seconds()),
    ).Int64()

    if err != nil {
        return 0, err
    }

    if result == -1 {
        return 0, ErrJobAlreadyRunning
    }

    return result, nil
}

// TryStartJob atomically tries to start a job (only if not already running)
func (s *JobScheduler) TryStartJob(ctx context.Context, jobName string, token int64) error {
    stateKey := "job:state:" + jobName

    // Use Lua script for atomic check-and-start
    script := redis.NewScript(`
        local state = redis.call('HGETALL', KEYS[1])
        if #state == 0 then
            return 'NOT_FOUND'
        end
        
        -- Convert HGETALL result to map
        local data = {}
        for i = 1, #state, 2 do
            data[state[i]] = state[i + 1]
        end
        
        local status = data['status']
        local currentToken = tonumber(data['fencing_token'])
        
        -- Check if job is already running by another instance with higher token
        if status == 'running' and currentToken > tonumber(ARGV[1]) then
            return 'STALE_TOKEN'
        end
        
        -- Check if job was completed recently (idempotency)
        if status == 'completed' then
            local completedAt = tonumber(data['completed_at'] or '0')
            local now = tonumber(ARGV[2])
            if now - completedAt < 300 then -- 5 minutes
                return 'ALREADY_DONE'
            end
        end
        
        -- Start the job
        redis.call('HSET', KEYS[1], 
            'status', 'running',
            'fencing_token', ARGV[1],
            'instance_id', ARGV[3],
            'started_at', ARGV[2],
            'completed_at', '')
        
        return 'OK'
    `)

    result, err := script.Run(ctx, s.redis,
        []string{stateKey},
        token, time.Now().Unix(), s.instanceID,
    ).Text()

    if err != nil {
        return err
    }

    switch result {
    case "OK":
        return nil
    case "NOT_FOUND":
        return ErrJobNotFound
    case "STALE_TOKEN":
        return ErrStaleToken
    case "ALREADY_DONE":
        return ErrJobAlreadyRunning
    default:
        return errors.New("unknown result: " + result)
    }
}

// CompleteJob marks job as completed
func (s *JobScheduler) CompleteJob(ctx context.Context, jobName string, token int64, success bool) error {
    stateKey := "job:state:" + jobName

    script := redis.NewScript(`
        local currentToken = tonumber(redis.call('HGET', KEYS[1], 'fencing_token') or '0')
        
        -- Only complete if we still own the job
        if currentToken ~= tonumber(ARGV[1]) then
            return 'STALE_TOKEN'
        end
        
        local status = 'completed'
        if ARGV[2] == 'false' then
            status = 'failed'
        end
        
        redis.call('HSET', KEYS[1], 
            'status', status,
            'completed_at', ARGV[3])
        
        return 'OK'
    `)

    result, err := script.Run(ctx, s.redis,
        []string{stateKey},
        token, strconv.FormatBool(success), time.Now().Unix(),
    ).Text()

    if err != nil {
        return err
    }

    if result == "STALE_TOKEN" {
        return ErrStaleToken
    }

    return nil
}

// ReleaseJobLock releases the lock (call after job completes or fails)
func (s *JobScheduler) ReleaseJobLock(ctx context.Context, jobName string, token int64) error {
    lockKey := "job:lock:" + jobName

    script := redis.NewScript(`
        if redis.call('GET', KEYS[1]) == ARGV[1] then
            return redis.call('DEL', KEYS[1])
        end
        return 0
    `)

    _, err := script.Run(ctx, s.redis,
        []string{lockKey}, s.instanceID,
    ).Result()

    return err
}
```

### Usage in Job Runner

```go
func (s *JobScheduler) RunDailyReport(ctx context.Context) error {
    jobName := "generate-daily-report"

    // 1. Acquire lock with fencing token
    token, err := s.AcquireJobLock(ctx, jobName, 5*time.Minute)
    if errors.Is(err, ErrJobAlreadyRunning) {
        log.Info("Job already running on another instance, skipping")
        return nil
    }
    if err != nil {
        return err
    }

    // 2. Try to start job (atomic check against stale executions)
    err = s.TryStartJob(ctx, jobName, token)
    if errors.Is(err, ErrStaleToken) {
        log.Info("Another instance started this job with higher token")
        return nil
    }
    if errors.Is(err, ErrJobAlreadyRunning) {
        log.Info("Job was already completed within last 5 minutes")
        return nil
    }
    if err != nil {
        return err
    }

    // 3. Do the actual work
    err = s.generateDailyReport(ctx)

    // 4. Mark job complete/failed
    success := err == nil
    if err != nil {
        s.CompleteJob(ctx, jobName, token, false)
        return err
    }
    s.CompleteJob(ctx, jobName, token, true)

    // 5. Release lock
    s.ReleaseJobLock(ctx, jobName, token)

    return nil
}
```

### Key Protections

| Problem | Protection |
|---------|------------|
| Pod OOM-killed, lock remains | Job state tracks running status; new instance sees "running" and skips |
| GC pause past TTL | Fencing token ensures only highest token can complete |
| Duplicate execution | Atomic TryStartJob with token check |
| Idempotency | Job state tracks completion; won't re-run within 5 min |

---

## Summary

| Aspect | Without Fencing | With Fencing |
|--------|-----------------|--------------|
| Safety | ❌ Zombie writes | ✅ Rejected by DB |
| Complexity | Simple | Moderate |
| Performance | Slighty better | Same |
| Correctness | Unreliable | Guaranteed |

**Remember:** The database is the source of truth, not the lock service.
