### **Problem:**

Developers often **don't know when to wrap an error (`%w`) or transform it (`%v`)**, leading to **unexpected coupling**, **lost error context**, or **unusable error handling**.

---

### **Solution:**

✅ **Use `%w` in `fmt.Errorf` when adding context but keeping the original error accessible.**  
✅ **Use `%v` in `fmt.Errorf` when transforming the error and preventing callers from unwrapping it.**  
✅ **Use custom error types when marking errors for specific handling.**

---

### **Best Practices:**

✅ **Wrap errors (`%w`) if callers should be able to inspect the original error.**  
✅ **Transform errors (`%v`) if the implementation details should remain hidden.**  
✅ **Use custom error types (`struct` or `var`) when marking errors for specific handling.**

---

### **Bad Example (Returning the Error Directly Without Context)**

```go
func Foo() error {
	err := bar()
	if err != nil {
		return err // ❌ No context, hard to debug
	}
	return nil
}
```

🔴 **Why is this wrong?**

- **No extra context** – The caller has no idea where the error originated.
- **Hard to debug** – If multiple functions call `bar()`, debugging becomes difficult.

---

### **Good Example (Using `%w` to Wrap the Error and Keep Context)**

```go
func Foo() error {
	err := bar()
	if err != nil {
		return fmt.Errorf("bar failed: %w", err) // ✅ Adds context while keeping the original error
	}
	return nil
}
```

🔵 **Why is this better?**

- **Keeps original error accessible via `errors.Is()` and `errors.As()`.**
- **Provides debugging context (`"bar failed"`) for better logs.**

---

### **Bad Example (Using `%v` Instead of `%w`, Making Error Unwrappable)**

```go
func Foo() error {
	err := bar()
	if err != nil {
		return fmt.Errorf("bar failed: %v", err) // ❌ Transforms error, making it unwrappable
	}
	return nil
}
```

🔴 **Why is this wrong?**

- **Prevents callers from using `errors.Is()` or `errors.As()`** to check for `bar()` errors.
- **Removes error context that might be useful for handling.**

---

### **Good Example (Using `%v` When We Want to Hide the Underlying Error)**

```go
func Foo() error {
	err := bar()
	if err != nil {
		return fmt.Errorf("operation failed: %v", err) // ✅ Transforms error, hiding details
	}
	return nil
}
```

🔵 **Why use this?**

- **Prevents callers from depending on internal error types, reducing coupling.**

---

### **Best Example (Using a Custom Error Type for Error Classification)**

```go
type ForbiddenError struct {
	Err error
}

func (e ForbiddenError) Error() string {
	return "forbidden: " + e.Err.Error()
}

func Foo() error {
	err := bar()
	if err != nil {
		return ForbiddenError{Err: err} // ✅ Clearly marks the error type
	}
	return nil
}
```

🔵 **Why is this useful?**

- **Makes it easy to check error types (`errors.As(err, &ForbiddenError{})`).**
- **Provides clear classification of errors without losing context.**

---

### **When to Use Each Approach:**

|**Scenario**|**Recommended Approach**|
|---|---|
|**Adding context to an error while keeping it accessible**|`fmt.Errorf("context: %w", err)`|
|**Transforming an error so callers can’t unwrap it**|`fmt.Errorf("new message: %v", err)`|
|**Marking an error for special handling**|Use a custom error type (`struct`)|
|**Returning an error as is**|Only if no extra context is needed|

---

### **Key Takeaways:**

- **Use `%w` in `fmt.Errorf` to wrap errors while keeping the original accessible.**
- **Use `%v` in `fmt.Errorf` when transforming an error to prevent unwrapping.**
- **Custom error types help classify errors for structured handling.**
- **Be mindful of error coupling—don’t expose internal errors unless necessary.** 🚀