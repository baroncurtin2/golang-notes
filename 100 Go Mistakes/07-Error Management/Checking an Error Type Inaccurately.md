### **Problem:**

Checking an error’s **type directly (`switch err.(type)`)** fails when errors are **wrapped using `%w`**, leading to incorrect error handling.

---

### **Solution:**

✅ **Use `errors.As()` to check if an error is of a specific type, even if wrapped.**  
✅ **Ensure the second argument to `errors.As()` is a pointer.**  
✅ **Prefer `errors.Is()` for comparing sentinel errors (fixed values).**

---

### **Best Practices:**

✅ **Use `errors.As()` when checking if an error belongs to a custom error type.**  
✅ **Use `errors.Is()` for direct comparison with known error values.**  
✅ **Avoid direct type assertions (`switch err.(type)`) unless you’re certain the error isn’t wrapped.**

---

### **Bad Example (Using Type Assertion Instead of `errors.As()`)**

```go
func handler(w http.ResponseWriter, r *http.Request) {
	amount, err := getTransactionAmount(transactionID)
	if err != nil {
		switch err.(type) { // ❌ Fails if `err` is wrapped
		case transientError:
			http.Error(w, err.Error(), http.StatusServiceUnavailable)
		default:
			http.Error(w, err.Error(), http.StatusBadRequest)
		}
		return
	}
}
```

🔴 **Why is this wrong?**

- **Fails when `err` is wrapped with `%w`** – Type assertion only works if the error is returned directly.
- **Returns the wrong status code (400 instead of 503) when `transientError` is wrapped.**

---

### **Good Example (Using `errors.As()` to Handle Wrapped Errors Correctly)**

```go
func handler(w http.ResponseWriter, r *http.Request) {
	amount, err := getTransactionAmount(transactionID)
	if err != nil {
		var te transientError
		if errors.As(err, &te) { // ✅ Unwraps the error and checks its type
			http.Error(w, err.Error(), http.StatusServiceUnavailable)
		} else {
			http.Error(w, err.Error(), http.StatusBadRequest)
		}
		return
	}
}
```

🔵 **Why is this better?**

- **Correctly identifies `transientError`, even if wrapped.**
- **Ensures `errors.As()` recursively unwraps errors to find the correct type.**

---

### **Bad Example (Returning a Wrapped Error but Checking it Incorrectly)**

```go
func getTransactionAmount(transactionID string) (float32, error) {
	amount, err := getTransactionAmountFromDB(transactionID)
	if err != nil {
		return 0, fmt.Errorf("failed to get transaction %s: %w", transactionID, err) // ✅ Correctly wraps error
	}
	return amount, nil
}

func getTransactionAmountFromDB(transactionID string) (float32, error) {
	return 0, transientError{err: errors.New("database issue")} // ✅ Creates a custom error type
}
```

🔴 **Why is this wrong?**

- **The handler expects `transientError`, but `getTransactionAmount()` wraps it.**
- **Type assertion (`switch err.(type)`) fails because `err` is wrapped.**

---

### **When to Use Each Approach:**

|**Scenario**|**Recommended Approach**|
|---|---|
|**Checking if an error is of a certain type**|Use `errors.As()`|
|**Checking if an error is a specific sentinel value**|Use `errors.Is()`|
|**Adding context while keeping the original error accessible**|Use `fmt.Errorf("message: %w", err)`|
|**Handling an error directly without wrapping**|Use direct comparison (`err == io.EOF`)|

---

### **Key Takeaways:**

- **Type assertions (`switch err.(type)`) fail when errors are wrapped with `%w`.**
- **Use `errors.As()` to check if an error belongs to a specific type, even if wrapped.**
- **Ensure the second argument in `errors.As()` is a pointer.**
- **For checking sentinel errors, use `errors.Is()` instead of type assertions.** 🚀