### **Problem:**

Developers often overlook **`sync.Cond`**, leading to **inefficient busy loops** or **missed notifications when multiple goroutines wait on an event**.

---

### **Solution:**

✅ **Use `sync.Cond` when multiple goroutines need to wait for a shared condition to change.**  
✅ **Use `Broadcast()` to wake all waiting goroutines when a condition is met.**  
✅ **Use `Signal()` if only one goroutine should be notified.**

---

### **Best Practices:**

✅ **Use `sync.Cond` when multiple goroutines need to be notified of a condition change.**  
✅ **Avoid busy loops that waste CPU cycles—use `Wait()` instead.**  
✅ **Ensure `Wait()` is called within a locked critical section (`mu.Lock()`).**  
✅ **Use `Broadcast()` when all waiting goroutines should be woken up.**  
✅ **Use `Signal()` when only one goroutine should proceed.**

---

### **Bad Example (Busy Loop Instead of Proper Synchronization)**

```go
type Donation struct {
	mu      sync.RWMutex
	balance int
}

func (d *Donation) WaitForGoal(goal int) {
	for d.balance < goal { // ❌ Inefficient busy loop
		time.Sleep(100 * time.Millisecond)
	}
	fmt.Printf("$%d goal reached\n", d.balance)
}

func (d *Donation) AddDonation(amount int) {
	d.mu.Lock()
	d.balance += amount
	d.mu.Unlock()
}
```

🔴 **Why is this wrong?**

- **Uses a busy loop** that constantly checks `balance`, wasting CPU cycles.
- **Inefficient for systems where multiple goroutines wait for updates.**

---

### **Good Example (Using `sync.Cond` to Signal Condition Changes Efficiently)**

```go
type Donation struct {
	mu      sync.Mutex
	cond    *sync.Cond
	balance int
}

func NewDonation() *Donation {
	d := &Donation{}
	d.cond = sync.NewCond(&d.mu) // ✅ Initializes condition variable
	return d
}

func (d *Donation) WaitForGoal(goal int) {
	d.mu.Lock()
	for d.balance < goal { // ✅ Waits efficiently without busy looping
		d.cond.Wait() // ✅ Releases lock while waiting, reacquires it when signaled
	}
	fmt.Printf("$%d goal reached\n", d.balance)
	d.mu.Unlock()
}

func (d *Donation) AddDonation(amount int) {
	d.mu.Lock()
	d.balance += amount
	d.mu.Unlock()
	d.cond.Broadcast() // ✅ Notifies all waiting goroutines
}
```

🔵 **Why is this better?**

- **No busy loop—goroutines wait efficiently without consuming CPU.**
- **Ensures synchronization—goroutines wait for `balance` to change.**
- **Broadcasts notifications to all waiting goroutines when `balance` updates.**

---

### **Best Example (Using `Signal()` to Wake Only One Waiting Goroutine)**

```go
func (d *Donation) WaitForSingleGoal(goal int) {
	d.mu.Lock()
	for d.balance < goal {
		d.cond.Wait() // ✅ Waits for a signal
	}
	fmt.Printf("Goal of $%d reached\n", d.balance)
	d.mu.Unlock()
}

func (d *Donation) AddDonation(amount int) {
	d.mu.Lock()
	d.balance += amount
	d.mu.Unlock()
	d.cond.Signal() // ✅ Wakes only one waiting goroutine
}
```

🔵 **Why is this useful?**

- **More efficient than `Broadcast()` when only one goroutine needs to proceed.**
- **Prevents unnecessary wake-ups, reducing contention.**

---

### **Comparison: `sync.Cond` vs. Channels for Signaling**

|**Scenario**|**Recommended Approach**|
|---|---|
|**Multiple goroutines waiting on a condition**|Use `sync.Cond` with `Broadcast()`|
|**Only one goroutine needs to be notified**|Use `sync.Cond` with `Signal()`|
|**Simple event notification without a condition check**|Use `chan struct{}`|
|**Workers processing messages asynchronously**|Use `chan T` instead|

---

### **Key Takeaways:**

- **`sync.Cond` is useful when multiple goroutines must wait for a condition to change.**
- **Use `Wait()` inside a locked section (`mu.Lock()`), which releases the lock while waiting.**
- **Use `Broadcast()` to wake all waiting goroutines when a condition changes.**
- **Use `Signal()` if only one goroutine needs to proceed.**
- **Avoid busy loops—use `sync.Cond` to wait efficiently.** 🚀