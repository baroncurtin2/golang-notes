### **Problem:**

Calling `wg.Add(1)` **inside a goroutine instead of the parent function** leads to **non-deterministic behavior** and can cause **incorrect wait group counts**, resulting in premature termination of `wg.Wait()`.

---

### **Solution:**

✅ **Always call `wg.Add(n)` in the parent function before launching goroutines.**  
✅ **Only call `wg.Done()` inside the goroutine.**  
✅ **Ensure `wg.Add(n)` and `wg.Wait()` establish a clear happens-before relationship.**

---

### **Best Practices:**

✅ **Call `wg.Add(n)` before starting `n` goroutines to ensure synchronization is correct.**  
✅ **Use `wg.Done()` inside each goroutine to decrement the counter properly.**  
✅ **Never call `wg.Add()` after `wg.Wait()` has already unblocked.**

---

### **Bad Example (Calling `wg.Add(1)` Inside the Goroutine, Causing Race Conditions)**

```go
var wg sync.WaitGroup
var v uint64

for i := 0; i < 3; i++ {
	go func() {
		wg.Add(1) // ❌ Called inside the goroutine
		atomic.AddUint64(&v, 1)
		wg.Done()
	}()
}

wg.Wait()
fmt.Println(v) // ❌ Non-deterministic output (0, 1, 2, or 3)
```

🔴 **Why is this wrong?**

- **`wg.Add(1)` may execute after `wg.Wait()` starts, causing incorrect synchronization.**
- **There’s no guarantee that all `wg.Add(1)` calls happen before `wg.Wait()` starts.**
- **`-race` flag will detect this as a data race.**

---

### **Good Example #1 (Calling `wg.Add(n)` Before Spawning Goroutines)**

```go
var wg sync.WaitGroup
var v uint64

wg.Add(3) // ✅ Ensures all goroutines are accounted for

for i := 0; i < 3; i++ {
	go func() {
		atomic.AddUint64(&v, 1)
		wg.Done() // ✅ Ensures proper decrement
	}()
}

wg.Wait()
fmt.Println(v) // ✅ Always prints 3
```

🔵 **Why is this better?**

- **Ensures `wg.Wait()` waits for exactly 3 goroutines.**
- **Eliminates race conditions between `wg.Add()` and `wg.Wait()`.**

---

### **Good Example #2 (Using `wg.Add(1)` Inside the Loop Instead of the Goroutine)**

```go
var wg sync.WaitGroup
var v uint64

for i := 0; i < 3; i++ {
	wg.Add(1) // ✅ Added before launching goroutine
	go func() {
		atomic.AddUint64(&v, 1)
		wg.Done()
	}()
}

wg.Wait()
fmt.Println(v) // ✅ Always prints 3
```

🔵 **Why is this better?**

- **Ensures that `wg.Add(1)` always happens before starting a goroutine.**
- **Still maintains correct synchronization.**

---

### **Best Example (Using a Worker Pool for Better Control and Performance)**

```go
var wg sync.WaitGroup
taskCh := make(chan int, 3)

for i := 0; i < 3; i++ {
	wg.Add(1)
	go func() {
		defer wg.Done()
		for task := range taskCh {
			fmt.Println("Processing:", task)
		}
	}()
}

for i := 0; i < 10; i++ {
	taskCh <- i
}
close(taskCh)

wg.Wait()
```

🔵 **Why is this the best approach?**

- **Uses a worker pool pattern to limit goroutines while still processing multiple tasks.**
- **Ensures proper synchronization with `wg.Wait()`.**
- **Avoids excessive goroutines and improves efficiency.**

---

### **When to Use Each Approach:**

|**Scenario**|**Recommended Approach**|
|---|---|
|**Launching a fixed number of goroutines**|Call `wg.Add(n)` before spawning goroutines|
|**Looping and launching goroutines dynamically**|Call `wg.Add(1)` inside the loop, not in the goroutine|
|**Processing a large number of tasks**|Use a worker pool with `sync.WaitGroup` and a channel|
|**Ensuring all goroutines finish before program exit**|Always use `wg.Wait()` properly|

---

### **Key Takeaways:**

- **Always call `wg.Add(n)` before launching `n` goroutines.**
- **Calling `wg.Add(1)` inside a goroutine can cause race conditions.**
- **Use worker pools for better control of concurrent workloads.**
- **Proper use of `sync.WaitGroup` ensures goroutines finish before `wg.Wait()` unblocks.** 🚀