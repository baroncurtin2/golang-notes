### **Problem:**

Using the `==` operator to compare errors **fails when the error is wrapped with `%w`**, leading to incorrect error handling and missed expected errors.

---

### **Solution:**

✅ **Use `errors.Is(err, ErrFoo)` instead of `err == ErrFoo` to check sentinel errors correctly.**  
✅ **Use sentinel errors (`var ErrFoo = errors.New("foo")`) for expected, checkable conditions.**  
✅ **Reserve custom error types (`struct{}`) for unexpected or structured errors.**

---

### **Best Practices:**

✅ **Use `errors.Is()` for comparing sentinel errors, as it works even if the error is wrapped.**  
✅ **Use `errors.New()` for sentinel errors when clients are expected to check for them.**  
✅ **Use `errors.As()` when you need to check for a custom error type instead of a sentinel value.**

---

### **Bad Example (Using `==` Instead of `errors.Is()`)**

```go
var ErrNoRows = errors.New("no rows found")

func query() error {
	return fmt.Errorf("query failed: %w", ErrNoRows) // ✅ Correctly wraps the error
}

func main() {
	err := query()
	if err != nil {
		if err == ErrNoRows { // ❌ Fails because `ErrNoRows` is wrapped
			fmt.Println("No rows found")
		} else {
			fmt.Println("Other error:", err)
		}
	}
}
```

🔴 **Why is this wrong?**

- **`err == ErrNoRows` fails** because `query()` wraps `ErrNoRows`, making direct comparison invalid.

---

### **Good Example (Using `errors.Is()` for Sentinel Error Comparison)**

```go
func main() {
	err := query()
	if err != nil {
		if errors.Is(err, ErrNoRows) { // ✅ Works even if the error is wrapped
			fmt.Println("No rows found")
		} else {
			fmt.Println("Other error:", err)
		}
	}
}
```

🔵 **Why is this better?**

- **`errors.Is()` correctly unwraps errors** and checks if `ErrNoRows` is present anywhere in the error chain.
- **Handles wrapped errors properly, preventing incorrect logic.**

---

### **Bad Example (Using Sentinel Errors for Unexpected Errors)**

```go
var ErrConnectionFailed = errors.New("database connection failed") // ❌ Sentinel error for unexpected case

func query() error {
	return ErrConnectionFailed // ❌ Unexpected errors should use error types
}
```

🔴 **Why is this wrong?**

- **Unexpected errors should not use sentinel values** because they often require additional context.
- **Using `errors.New()` limits the ability to store structured data.**

---

### **Good Example (Using Custom Error Types for Unexpected Errors)**

```go
type ConnectionError struct {
	Host string
}

func (e *ConnectionError) Error() string {
	return fmt.Sprintf("failed to connect to %s", e.Host)
}

func query() error {
	return &ConnectionError{Host: "db.local"} // ✅ Uses structured error type
}
```

🔵 **Why is this better?**

- **Allows storing extra details (`Host`) for debugging.**
- **Avoids treating unexpected errors the same way as expected errors.**

---

### **When to Use Each Approach:**

|**Scenario**|**Recommended Approach**|
|---|---|
|**Checking if an error matches a known sentinel error**|Use `errors.Is(err, ErrFoo)`|
|**Checking if an error belongs to a specific type**|Use `errors.As(err, &CustomError{})`|
|**Defining expected errors that clients should check**|Use `var ErrFoo = errors.New("foo")`|
|**Handling unexpected or structured errors**|Use a custom error type (`struct{}`)|

---

### **Key Takeaways:**

- **Use `errors.Is()` instead of `==` for checking sentinel errors.**
- **Use sentinel errors (`errors.New()`) for expected conditions (e.g., `sql.ErrNoRows`).**
- **Use custom error types (`struct{}`) for unexpected errors requiring more context.**
- **Avoid using sentinel errors for cases where additional information is needed.** 🚀