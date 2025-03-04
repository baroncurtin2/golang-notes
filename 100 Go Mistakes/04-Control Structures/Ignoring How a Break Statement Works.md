### **Problem:**

When using `break` inside **nested loops, switch, or select statements**, developers often mistakenly **break the wrong statement**, leading to unexpected behavior.

---

### **Solution:**

✅ **Understand that `break` affects the innermost enclosing loop, switch, or select statement.**  
✅ **Use labeled `break` to exit the correct loop when multiple nested structures exist.**

---

### **Best Practices:**

✅ **Use labels (`break label`) to clarify intent when breaking an outer loop.**  
✅ **Be mindful that `break` inside `switch` and `select` does NOT exit the loop.**  
✅ **Use meaningful labels that describe the purpose of the loop.**

---

### **Bad Example (Break Only Exits `switch`, Not `for` Loop)**

```go
for i := 0; i < 5; i++ {
	fmt.Printf("%d ", i)
	switch i {
	case 2:
		break // ❌ Only exits the switch, NOT the for loop
	}
}
fmt.Println() // Output: 0 1 2 3 4 ❌ Loop does NOT break at 2
```

🔴 **Why is this wrong?**

- `break` **only exits the `switch` statement**, NOT the enclosing `for` loop.
- The loop **continues running until `i < 5` is false**.

---

### **Good Example (Using a Labeled Break to Exit the Loop)**

```go
loop:
for i := 0; i < 5; i++ {
	fmt.Printf("%d ", i)
	switch i {
	case 2:
		break loop // ✅ Exits the entire for loop
	}
}
fmt.Println() // Output: 0 1 2 ✅ Loop stops correctly
```

🔵 **Why is this better?**

- **Uses a labeled break (`break loop`)** to exit the outer `for` loop.
- **Ensures the loop stops immediately when `i == 2`**.

---

### **Bad Example (Breaking `select` Instead of the `for` Loop)**

```go
for {
	select {
	case <-ch:
		// Process data
	case <-ctx.Done():
		break // ❌ Only exits `select`, NOT the loop
	}
}
```

🔴 **Why is this wrong?**

- **Break exits only the `select` statement, NOT the enclosing `for` loop.**
- **Loop continues running, causing unexpected behavior.**

---

### **Good Example (Using a Labeled Break for `for-select` Loops)**

```go
loop:
for {
	select {
	case <-ch:
		// Process data
	case <-ctx.Done():
		break loop // ✅ Exits the entire for loop
	}
}
```

🔵 **Why is this better?**

- **Ensures `break` exits the `for` loop, not just the `select` statement.**

---

### **Key Takeaways:**

- **`break` exits only the innermost `for`, `switch`, or `select` statement.**
- **Use labeled `break` (`break label`) to exit an outer loop.**
- **Be cautious when using `break` in `select` inside a loop—it does NOT exit the loop.** 🚀