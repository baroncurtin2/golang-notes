### **Problem:**

Starting a **goroutine without a defined stopping mechanism** leads to **goroutine leaks**, resulting in **memory bloat, resource exhaustion (e.g., open DB connections, file descriptors), and unpredictable behavior**.

---

### **Solution:**

✅ **Ensure every goroutine has a defined stop condition.**  
✅ **Use `context.Context` to signal cancellation for long-running goroutines.**  
✅ **Use `sync.WaitGroup` to ensure goroutines complete before program exit.**  
✅ **Close channels properly to avoid leaking goroutines waiting on them.**

---

### **Best Practices:**

✅ **Pass a `context.Context` to goroutines so they can listen for cancellation.**  
✅ **Ensure a goroutine stops when its job is done (e.g., through channel closure).**  
✅ **Use `defer` to ensure cleanup functions (like closing files, DB connections) are always executed.**

---

### **Bad Example (Goroutine Leak: No Stop Condition)**

```go
func startWorker() chan int {
	ch := make(chan int)
	go func() {
		for val := range ch { // ❌ Goroutine will never exit if `ch` is never closed
			fmt.Println(val)
		}
	}()
	return ch
}

func main() {
	ch := startWorker()
	ch <- 1
	// No call to close(ch), goroutine leaks forever
}
```

🔴 **Why is this wrong?**

- **If `ch` is never closed, the goroutine will be stuck in an infinite loop.**
- **The program may exit, but the goroutine stays alive in the background, consuming resources.**

---

### **Good Example (Using `context.Context` to Stop a Goroutine Gracefully)**

```go
func startWorker(ctx context.Context) chan int {
	ch := make(chan int)
	go func() {
		defer fmt.Println("Worker stopped") // ✅ Ensures cleanup on exit
		for {
			select {
			case val := <-ch:
				fmt.Println(val)
			case <-ctx.Done(): // ✅ Stops when context is canceled
				return
			}
		}
	}()
	return ch
}

func main() {
	ctx, cancel := context.WithCancel(context.Background())
	ch := startWorker(ctx)

	ch <- 1
	time.Sleep(time.Second)

	cancel() // ✅ Stops the worker goroutine
	time.Sleep(time.Second) // Give time for cleanup
}
```

🔵 **Why is this better?**

- **Uses `context.Context` to send a cancellation signal.**
- **Ensures the goroutine stops when no longer needed.**
- **Logs when the worker stops, making debugging easier.**

---

### **Best Example (Using `sync.WaitGroup` to Ensure Goroutine Completion Before Exit)**

```go
func startWorker(wg *sync.WaitGroup, ch <-chan int) {
	defer wg.Done() // ✅ Ensures the WaitGroup counter is decremented
	for val := range ch {
		fmt.Println("Processed:", val)
	}
	fmt.Println("Worker stopped") // ✅ Ensures we see when the worker exits
}

func main() {
	var wg sync.WaitGroup
	ch := make(chan int)

	wg.Add(1)
	go startWorker(&wg, ch)

	ch <- 1
	close(ch) // ✅ Closing the channel stops the worker
	wg.Wait() // ✅ Ensures the program doesn't exit until the worker is done
}
```

🔵 **Why is this the best?**

- **Uses `sync.WaitGroup` to ensure the program doesn't exit before the goroutine finishes.**
- **Closes the channel to signal the worker to stop gracefully.**
- **No leaked goroutines; ensures cleanup.**

---

### **When to Use Each Approach:**

|**Scenario**|**Recommended Solution**|
|---|---|
|**Long-running goroutine that should stop on demand**|Use `context.WithCancel()`|
|**Ensuring all goroutines finish before program exit**|Use `sync.WaitGroup`|
|**Worker processing jobs from a channel**|Ensure the channel is closed to stop workers|
|**One-time background task**|Use `defer` and ensure no infinite loops|

---

### **Key Takeaways:**

- **Every goroutine must have a defined stopping condition—never start one without a plan to stop it.**
- **Use `context.Context` for cancellation signals in long-running goroutines.**
- **Use `sync.WaitGroup` to ensure goroutines finish before program exit.**
- **Close channels properly to prevent goroutines from blocking indefinitely.** 🚀