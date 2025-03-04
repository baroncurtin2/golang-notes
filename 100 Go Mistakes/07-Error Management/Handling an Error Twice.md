### **Problem:**

Handling an error **multiple times** (e.g., **logging and returning it**) results in **duplicate logs, unnecessary complexity, and harder debugging**, especially in concurrent applications.

---

### **Solution:**

✅ **Handle each error only once** – Either log it or return it, but not both.  
✅ **Use error wrapping (`fmt.Errorf("%w")`) to add context instead of logging multiple times.**  
✅ **Ensure error messages contain enough context so logs remain meaningful at higher levels.**

---

### **Best Practices:**

✅ **Avoid redundant logging** – Logging an error **before returning it** results in duplicate messages.  
✅ **Use `fmt.Errorf("%w", err)` to add context instead of logging and returning separately.**  
✅ **Ensure errors are only handled at the necessary level** – The caller should log the error, not intermediate functions.

---

### **Bad Example (Logging and Returning the Same Error, Causing Duplicate Logs)**

```go
func validateCoordinates(lat, lng float32) error {
	if lat > 90.0 || lat < -90.0 {
		log.Printf("invalid latitude: %f", lat) // ❌ Logs the error
		return fmt.Errorf("invalid latitude: %f", lat) // ❌ Also returns the error
	}
	if lng > 180.0 || lng < -180.0 {
		log.Printf("invalid longitude: %f", lng) // ❌ Logs again
		return fmt.Errorf("invalid longitude: %f", lng) // ❌ Returns again
	}
	return nil
}

func GetRoute(srcLat, srcLng, dstLat, dstLng float32) (Route, error) {
	err := validateCoordinates(srcLat, srcLng)
	if err != nil {
		log.Println("failed to validate source coordinates") // ❌ Logs again
		return Route{}, err
	}

	err = validateCoordinates(dstLat, dstLng)
	if err != nil {
		log.Println("failed to validate target coordinates") // ❌ Logs again
		return Route{}, err
	}

	return getRoute(srcLat, srcLng, dstLat, dstLng)
}
```

🔴 **Why is this wrong?**

- **Logs the same error multiple times** – Once inside `validateCoordinates`, then again in `GetRoute`.
- **Creates redundant log messages**, making debugging harder in concurrent applications.
- **Increases noise in logs**, making it difficult to identify unique failures.

---

### **Good Example (Returning Errors Without Logging Multiple Times)**

```go
func validateCoordinates(lat, lng float32) error {
	if lat > 90.0 || lat < -90.0 {
		return fmt.Errorf("invalid latitude: %f", lat) // ✅ Only returns error
	}
	if lng > 180.0 || lng < -180.0 {
		return fmt.Errorf("invalid longitude: %f", lng) // ✅ Only returns error
	}
	return nil
}

func GetRoute(srcLat, srcLng, dstLat, dstLng float32) (Route, error) {
	err := validateCoordinates(srcLat, srcLng)
	if err != nil {
		return Route{}, fmt.Errorf("failed to validate source coordinates: %w", err) // ✅ Wraps error with context
	}

	err = validateCoordinates(dstLat, dstLng)
	if err != nil {
		return Route{}, fmt.Errorf("failed to validate target coordinates: %w", err) // ✅ Wraps error with context
	}

	return getRoute(srcLat, srcLng, dstLat, dstLng)
}
```

🔵 **Why is this better?**

- **Avoids redundant logging** – The error is only logged once when handled at the top level.
- **Uses `%w` to wrap the error**, providing **additional context while preserving the original error.**
- **Ensures logs remain meaningful without excessive duplication.**

---

### **Best Example (Logging at the Right Level Without Duplication)**

```go
func main() {
	route, err := GetRoute(200.0, 50.0, 30.0, 20.0)
	if err != nil {
		log.Printf("error getting route: %v", err) // ✅ Logs error only once at the top level
		return
	}
	fmt.Println("Route:", route)
}
```

🔵 **Why is this best?**

- **Only logs at the highest level where an error is actionable.**
- **Ensures logs remain meaningful and not repetitive.**
- **Allows error propagation while keeping logs clean.**

---

### **When to Use Each Approach:**

|**Scenario**|**Recommended Approach**|
|---|---|
|**Returning an error from a function**|Do not log, just return the error|
|**Adding context to an error before returning**|Use `fmt.Errorf("%w", err)`|
|**Logging an error**|Do it only at the highest level where action is taken|
|**Handling an error without returning**|Log it if necessary|

---

### **Key Takeaways:**

- **Never log and return the same error** – This causes duplicate logs and clutter.
- **Use error wrapping (`%w`) to add context instead of logging multiple times.**
- **Log errors only at the highest level where action is taken.**
- **Ensure logs remain meaningful by avoiding redundant messages.** 🚀