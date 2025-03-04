### **Problem:**

Starting goroutines inside a loop without properly **capturing loop variables** leads to **unexpected and non-deterministic behavior**, where all goroutines may reference the **same, changing variable** instead of a fixed value.

---

### **Solution:**

✅ **Capture loop variables inside a local variable before launching the goroutine.**  
✅ **Use function parameters to pass the correct value to the goroutine.**

---

### **Best Practices:**

✅ **Avoid referencing loop variables directly inside goroutine closures.**  
✅ **Use local variables inside loops to ensure goroutines get the correct value.**  
✅ **Use function parameters to pass loop values explicitly.**

---

### **Bad Example (Loop Variable Captured Incorrectly in a Goroutine)**

```go
s := []int{1, 2, 3}

for _, i := range s {
	go func() {
		fmt.Print(i) // ❌ Captures `i` incorrectly, may print unexpected values
	}()
}
```

🔴 **Why is this wrong?**

- **All goroutines share the same `i` variable**, so they may print unexpected values.
- **By the time goroutines execute, `i` may have changed.**

**Possible outputs:**

```
233
333
122
```

(Non-deterministic behavior)

---

### **Good Example #1 (Using a Local Variable to Capture the Correct Value)**

```go
for _, i := range s {
	val := i // ✅ Captures `i` in a local variable
	go func() {
		fmt.Print(val) // ✅ Uses the fixed `val`
	}()
}
```

🔵 **Why is this better?**

- **Each goroutine gets a unique copy of the value, preventing race conditions.**
- **Ensures the correct value is used when the goroutine executes.**

---

### **Good Example #2 (Using Function Parameters to Capture Values)**

```go
for _, i := range s {
	go func(val int) {
		fmt.Print(val) // ✅ `val` is passed explicitly as a parameter
	}(i) // ✅ Pass `i` to the function immediately
}
```

🔵 **Why is this better?**

- **No need for an extra variable—captures `i` immediately via function arguments.**
- **Ensures that goroutines get the intended value of `i`.**

---

### **When to Use Each Approach:**

|**Scenario**|**Recommended Approach**|
|---|---|
|**Keeping goroutines inside a loop**|Use a local variable (`val := i`)|
|**Avoiding closures capturing loop variables**|Pass values via function parameters (`func(val int)`)|
|**Iterating over large datasets with concurrency**|Use worker pools instead of spawning too many goroutines|

---

### **Key Takeaways:**

- **Loop variables inside goroutines are shared across iterations—always capture them locally.**
- **Using a local variable (`val := i`) ensures each goroutine gets the correct value.**
- **Passing loop variables as function parameters eliminates unintended sharing.**
- **Both solutions are valid—choose based on readability and style preference.** 🚀