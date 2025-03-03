### **Problem:**

When using a `range` loop in Go, the **expression is evaluated only once**, meaning that **any modifications to the original data structure after the loop starts are ignored**. This can lead to **unexpected behavior when appending to slices, changing loop variables, or modifying arrays inside the loop**.

---

### **Solution:**

✅ **Remember that `range` creates a copy of arrays and slices before iterating.**  
✅ **Use explicit indexing (`slice[i]`) when modifying slices during iteration.**  
✅ **For arrays, use pointers (`&array`) if modifications should persist.**  
✅ **Avoid changing the iterating expression (`slice = append(slice, x)`) inside a `range` loop.**

---

### **Best Practices:**

✅ **Do not modify the iterating slice inside a `range` loop** – Changes will not affect the loop.  
✅ **Use explicit indexing (`for i := range slice`) if you need to update elements.**  
✅ **Use array pointers (`&array`) when iterating over arrays that need to be modified.**

---

### **Bad Example (Appending to Slice in a `range` Loop, Doesn't Grow the Loop)**

```go
s := []int{0, 1, 2}

for range s {
	s = append(s, 10) // ❌ The loop runs only 3 times because range evaluates s once
}
fmt.Println(s) // Output: [0 1 2 10 10 10] (slice grows but loop doesn’t iterate over new elements)
```

🔴 **Why is this wrong?**

- `range` **evaluates `s` once at the start** (length = 3), so the loop **only iterates 3 times**, even though `append()` grows the slice.

---

### **Good Example (Using `for i := 0; i < len(slice); i++`)**

```go
s := []int{0, 1, 2}

for i := 0; i < len(s); i++ {
	s = append(s, 10) // ✅ Loop continues as slice grows
}
```

🔵 **Why is this better?**

- `len(s)` **is evaluated on every iteration**, allowing new elements to be processed.

---

### **Bad Example (Modifying an Array in a `range` Loop Doesn't Update the Copy)**

```go
a := [3]int{0, 1, 2}

for i, v := range a {
	a[2] = 10 // ❌ Updates original array, but `v` is from the copy
	if i == 2 {
		fmt.Println(v) // ❌ Still prints 2, not 10
	}
}
```

🔴 **Why is this wrong?**

- `range` **copies the array** before iteration, so `v` is never updated even if `a[2]` is changed.

---

### **Good Example (Using Array Pointers to Modify in `range`)**

```go
a := [3]int{0, 1, 2}

for i, v := range &a { // ✅ Uses array pointer to avoid copying
	a[2] = 10
	if i == 2 {
		fmt.Println(v) // ✅ Now prints 10
	}
}
```

🔵 **Why is this better?**

- **Iterates over a pointer (`&a`)**, so `v` reflects the actual array updates.

---

### **Bad Example (Switching Channels in a `range` Loop Has No Effect)**

```go
ch1 := make(chan int, 3)
go func() {
	ch1 <- 0
	ch1 <- 1
	ch1 <- 2
	close(ch1)
}()

ch2 := make(chan int, 3)
go func() {
	ch2 <- 10
	ch2 <- 11
	ch2 <- 12
	close(ch2)
}()

ch := ch1
for v := range ch { // ❌ `range` evaluated `ch1` once, never switches to `ch2`
	fmt.Println(v)
	ch = ch2
}
```

🔴 **Why is this wrong?**

- `range` **evaluates `ch1` at the start** and does not switch to `ch2`, so `ch2` is ignored.

---

### **When to Be Cautious with `range`:**

✅ When **appending to slices** inside the loop (use `for i < len(slice)` instead).  
✅ When **modifying array elements inside a `range` loop** (use array pointers).  
✅ When **switching between channels** during iteration (use `select` instead).

---

### **Key Takeaways:**

- **`range` evaluates the iterating expression once before the loop starts.**
- **Appending inside a `range` loop does not grow the iteration count.**
- **Arrays are copied in `range`, so modifications inside the loop won’t reflect in the loop variable.**
- **Use pointers (`&array`) to modify arrays inside `range` loops.** 🚀