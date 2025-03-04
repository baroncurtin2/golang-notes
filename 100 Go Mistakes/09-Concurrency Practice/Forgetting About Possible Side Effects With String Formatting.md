### **Problem:**

String formatting (`fmt.Sprintf`, `fmt.Errorf`) in concurrent applications can cause **unexpected side effects**, such as **data races and deadlocks**, due to implicit method calls like `String()`.

---

### **Solution:**

✅ **Avoid using `fmt.Sprintf` or `fmt.Errorf` inside locked sections if they trigger other methods that acquire locks.**  
✅ **Restrict the scope of mutex locks to avoid unnecessary locking during error handling.**  
✅ **Be aware that `fmt.Sprintf("%v", obj)` calls `obj.String()`, which may introduce side effects in concurrent code.**

---

### **Best Practices:**

✅ **Use logging without struct formatting inside locked sections to avoid deadlocks.**  
✅ **Always test concurrent code with negative scenarios to detect deadlocks early.**  
✅ **If struct formatting is required, copy values before unlocking the mutex.**

---

### **Bad Example #1 (Data Race Due to `fmt.Sprintf` with Context Values)**

```go
func (w *watcher) Watch(ctx context.Context, key string) {
	ctxKey := fmt.Sprintf("%v", ctx) // ❌ Reads from a possibly mutable context value
	w.streams[ctxKey] = newStream()
}
```

🔴 **Why is this wrong?**

- **`fmt.Sprintf("%v", ctx)` traverses all context values, potentially reading mutable data concurrently.**
- **If another goroutine modifies `ctx` while `fmt.Sprintf` is executing, it leads to a data race.**

✅ **Fix:** Avoid string formatting that involves reading from shared or mutable state.

```go
func (w *watcher) Watch(ctx context.Context, key string) {
	ctxKey := extractKey(ctx) // ✅ Extract a stable key instead of formatting the entire context
	w.streams[ctxKey] = newStream()
}
```

---

### **Bad Example #2 (Deadlock Due to `fmt.Sprintf` Calling `String()` on a Locked Struct)**

```go
type Customer struct {
	mutex sync.RWMutex
	id    string
	age   int
}

func (c *Customer) UpdateAge(age int) error {
	c.mutex.Lock() // ✅ Locks struct
	defer c.mutex.Unlock()

	if age < 0 {
		return fmt.Errorf("age should be positive for customer %v", c) // ❌ Calls `c.String()` while locked
	}

	c.age = age
	return nil
}

func (c *Customer) String() string {
	c.mutex.RLock() // ❌ Tries to acquire read lock while write lock is held
	defer c.mutex.RUnlock()
	return fmt.Sprintf("id %s, age %d", c.id, c.age)
}
```

🔴 **Why is this wrong?**

- **`fmt.Errorf("%v", c)` triggers `c.String()`, which acquires a read lock while a write lock is already held.**
- **This results in a deadlock—`UpdateAge` holds a write lock while `String()` waits for a read lock.**

✅ **Fix:** Restrict the scope of the lock and avoid formatting the entire struct inside the locked section.

```go
func (c *Customer) UpdateAge(age int) error {
	if age < 0 {
		return fmt.Errorf("age should be positive for customer id %s", c.id) // ✅ Avoids calling `String()`
	}

	c.mutex.Lock()
	defer c.mutex.Unlock()
	c.age = age
	return nil
}
```

🔵 **Why is this better?**

- **Locks the mutex only after validation.**
- **Avoids calling `String()` while the struct is locked, preventing deadlocks.**

---

### **Best Example (Copying Values Before Unlocking the Mutex for Safe String Formatting)**

```go
func (c *Customer) String() string {
	c.mutex.RLock()
	id := c.id
	age := c.age
	c.mutex.RUnlock() // ✅ Unlocks before calling `fmt.Sprintf`

	return fmt.Sprintf("id %s, age %d", id, age)
}
```

🔵 **Why is this the best approach?**

- **Captures struct values inside the lock and releases it before formatting the string.**
- **Ensures that `String()` never triggers a deadlock by waiting on an already locked mutex.**

---

### **When to Be Careful with String Formatting in Concurrent Code:**

|**Scenario**|**Potential Issue**|**Recommended Fix**|
|---|---|---|
|**Formatting a struct with `fmt.Sprintf("%v", obj)`**|Calls `String()`, which may acquire locks|Unlock before formatting or log only immutable fields|
|**Using `fmt.Errorf("%v", ctx)` in a concurrent context**|Reads from possibly mutable state|Extract a stable key from the context instead|
|**Logging data from a mutex-locked struct**|Can cause deadlocks|Copy values before unlocking|

---

### **Key Takeaways:**

- **Be careful when using `fmt.Sprintf("%v", obj)` in concurrent code—it may trigger struct methods like `String()`, leading to unexpected behavior.**
- **Avoid calling `fmt.Errorf("%v", obj)` inside a locked section if `obj.String()` acquires a lock.**
- **For concurrent-safe struct formatting, copy values before unlocking the mutex.**
- **Always test concurrency scenarios that involve logging, error formatting, or struct serialization.** 🚀