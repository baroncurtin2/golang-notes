### **Problem:**

Type embedding in Go can **unintentionally expose fields and methods**, leading to **unexpected behaviors and breaking encapsulation**. This is especially problematic when embedding types like `sync.Mutex`, which should remain private.

---

### **Solution:**

✅ **Use embedding only when it provides clear benefits**, such as promoting behavior that should be publicly accessible.  
✅ **Avoid embedding when it exposes internal details** – If a field should remain private, store it as a named field instead of embedding it.  
✅ **Favor composition over embedding when encapsulation is required.**

---

### **Best Practices:**

✅ **Use embedding to promote meaningful methods** – e.g., embedding `io.Writer` to avoid boilerplate forwarding methods.  
✅ **Avoid embedding when it exposes methods that should remain private** – e.g., `sync.Mutex`.  
✅ **Don’t use embedding just to shorten access to fields** – e.g., replacing `Foo.Bar.Baz` with `Foo.Baz` isn’t a strong reason.  
✅ **Be cautious when embedding in exported structs** – Future changes to the embedded type may introduce unintended method promotions.

---

### **Bad Example (Embedding Exposes Unintended Methods)**

```go
package store

import "sync"

type InMem struct {
    sync.Mutex  // ❌ Embedding exposes Lock/Unlock methods to external users
    m map[string]int
}

func (i *InMem) Get(key string) (int, bool) {
    i.Lock()
    v, contains := i.m[key]
    i.Unlock()
    return v, contains
}

func New() *InMem {
    return &InMem{m: make(map[string]int)}
}

func main() {
    memStore := New()
    memStore.Lock() // ❌ External users can lock/unlock, breaking encapsulation
}
```

🔴 **Why is this wrong?**

- `sync.Mutex` should be encapsulated; external users should not have direct access to `Lock/Unlock`.
- Exposes unnecessary complexity to consumers.

---

### **Good Example (Encapsulating the Mutex Properly)**

```go
package store

import "sync"

type InMem struct {
    mu sync.Mutex  // ✅ Mutex is private, preventing external access
    m  map[string]int
}

func (i *InMem) Get(key string) (int, bool) {
    i.mu.Lock()
    defer i.mu.Unlock()
    v, contains := i.m[key]
    return v, contains
}
```

🔵 **Why is this better?**

- **Encapsulation maintained** – Mutex methods are not accessible outside the struct.
- **Safer API** – Prevents unintended locking behavior by external users.

---

### **Good Example (Using Embedding Correctly for Behavior Promotion)**

```go
package logger

import (
    "io"
    "os"
)

type Logger struct {
    io.WriteCloser  // ✅ Embedding promotes Write and Close methods
}

func NewLogger() *Logger {
    return &Logger{WriteCloser: os.Stdout}
}

func main() {
    log := NewLogger()
    log.Write([]byte("Hello"))  // ✅ No need to manually forward Write method
    log.Close()
}
```

🔵 **Why is this good?**

- **Avoids boilerplate** – No need to manually implement `Write` and `Close` methods.
- **Maintains correct encapsulation** – `io.WriteCloser` behavior is intentionally exposed.

---

### **When to Use Type Embedding:**

✅ **To compose behavior without manually forwarding methods** (e.g., embedding `io.Writer`).  
✅ **To satisfy interfaces without extra boilerplate** (e.g., `io.ReadWriter`).

### **When to Avoid Type Embedding:**

❌ **If it exposes private fields or methods unintentionally** (e.g., `sync.Mutex`).  
❌ **If it’s used purely to shorten field access** (e.g., replacing `Foo.Bar.Baz` with `Foo.Baz`).  
❌ **If it increases maintenance effort for exported structs** (future changes in embedded types may introduce unintended behaviors).

---

### **Key Takeaways:**

- **Use embedding for behavior, not just field access.**
- **Avoid embedding when encapsulation is required.**
- **Be mindful of exposing unintended methods in public APIs.**
- **Favor composition over embedding when in doubt.** 🚀