### **Problem:**

Developers **misunderstand or misuse Go’s `context.Context`**, leading to **unresponsive applications, resource leaks, and poor cancellation handling in concurrent code**.

---

### **Solution:**

✅ **Use `context.WithTimeout` or `context.WithDeadline` for operations that should time out.**  
✅ **Use `context.WithCancel` to allow manual cancellation of running goroutines.**  
✅ **Pass `context.Context` to all long-running functions to support graceful cancellation.**  
✅ **Use `ctx.Done()` in select statements to avoid blocking indefinitely.**

---

### **Best Practices:**

✅ **Always propagate context from the caller instead of creating a new `context.Background()`.**  
✅ **Use `defer cancel()` when using `context.WithTimeout()` to avoid memory leaks.**  
✅ **Use `context.WithValue()` only for request-scoped metadata (e.g., request IDs, tracing).**  
✅ **Ensure all goroutines respect context cancellation (`<-ctx.Done()`).**

---

### **Bad Example (Creating a New Context Instead of Propagating One)**

```go
func fetchData() {
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second) // ❌ Creates a new context
	defer cancel()

	doRequest(ctx)
}
```

🔴 **Why is this wrong?**

- **The function should accept a `context.Context` parameter to propagate cancellation from its caller.**
- **Forces every function to define its own timeout, reducing flexibility.**

---

### **Good Example (Propagating Context Instead of Creating a New One)**

```go
func fetchData(ctx context.Context) {
	ctx, cancel := context.WithTimeout(ctx, 5*time.Second) // ✅ Propagates caller’s context
	defer cancel()

	doRequest(ctx)
}
```

🔵 **Why is this better?**

- **Allows the caller to control timeouts and cancellations.**
- **Ensures consistent context propagation across function calls.**

---

### **Bad Example (Ignoring Context Cancellation in a Goroutine)**

```go
func worker() {
	for {
		processTask() // ❌ Runs indefinitely, even if the program is shutting down
	}
}
```

🔴 **Why is this wrong?**

- **The goroutine runs forever, even if the main function exits.**
- **No way to cancel it gracefully.**

---

### **Good Example (Handling Context Cancellation in a Goroutine)**

```go
func worker(ctx context.Context) {
	for {
		select {
		case <-ctx.Done(): // ✅ Stops when context is canceled
			fmt.Println("Worker shutting down...")
			return
		default:
			processTask()
		}
	}
}
```

🔵 **Why is this better?**

- **Properly listens for cancellation signals using `ctx.Done()`.**
- **Allows the program to shut down cleanly.**

---

### **Bad Example (Blocking on a Channel Without Checking Context Cancellation)**

```go
func sendMessage(ch chan string) {
	ch <- "Hello" // ❌ Blocks forever if no one is receiving
}
```

🔴 **Why is this wrong?**

- **If no goroutine reads from `ch`, this will deadlock.**
- **No way to cancel the operation if needed.**

---

### **Good Example (Using `select` to Handle Both Channel and Context Cancellation)**

```go
func sendMessage(ctx context.Context, ch chan string) {
	select {
	case ch <- "Hello": // ✅ Sends if possible
	case <-ctx.Done():  // ✅ Cancels if context is done
		fmt.Println("Send canceled")
	}
}
```

🔵 **Why is this better?**

- **Prevents blocking indefinitely if the receiver is not ready.**
- **Ensures graceful cancellation when the context is canceled.**

---

### **Bad Example (Misusing `context.WithValue()` for Business Logic)**

```go
func handler(ctx context.Context) {
	ctx = context.WithValue(ctx, "userID", 123) // ❌ Using `WithValue()` for business logic
	process(ctx)
}
```

🔴 **Why is this wrong?**

- **`context.WithValue()` should only be used for request-scoped metadata (e.g., tracing, authentication).**
- **It is not meant for passing application data.**

---

### **Good Example (Using `context.WithValue()` for Tracing or Request Metadata)**

```go
type contextKey string

const requestIDKey contextKey = "requestID"

func handler(ctx context.Context) {
	ctx = context.WithValue(ctx, requestIDKey, "abc-123") // ✅ Using it for request-scoped metadata
	process(ctx)
}

func process(ctx context.Context) {
	if reqID, ok := ctx.Value(requestIDKey).(string); ok {
		fmt.Println("Processing request:", reqID)
	}
}
```

🔵 **Why is this better?**

- **Uses `context.WithValue()` correctly for request-scoped metadata (tracing, request IDs).**
- **Prevents key collisions by using a custom type (`contextKey`).**

---

### **When to Use Each Context Type:**

|**Scenario**|**Recommended Approach**|
|---|---|
|**Cancel an operation manually**|`context.WithCancel()`|
|**Set a deadline for an operation**|`context.WithDeadline()`|
|**Set a timeout for an operation**|`context.WithTimeout()`|
|**Pass request-scoped metadata**|`context.WithValue()` (use sparingly)|
|**Check if a context is canceled**|Use `ctx.Done()` in `select`|

---

### **Key Takeaways:**

- **Always pass `context.Context` to functions that perform long-running tasks.**
- **Use `context.WithTimeout()` to prevent operations from running indefinitely.**
- **Check `ctx.Done()` inside goroutines to ensure they stop when canceled.**
- **Use `context.WithValue()` only for request metadata, not business logic.** 🚀