### **Problem:**

When using `range` loops with **pointer elements**, the **loop variable is reused across iterations**, meaning all stored pointers reference the **same memory location**, which **unexpectedly points to the last element**.

---

### **Solution:**

✅ **Create a new local variable inside the loop** – Ensures each pointer references a distinct object.  
✅ **Use index-based iteration (`customers[i]`)** – Directly references elements in the original slice.

---

### **Best Practices:**

✅ **Avoid storing pointers to loop variables** – They reference the same memory address.  
✅ **Use `customers[i]` instead of `&customer` inside `range` loops** – Ensures correct references.  
✅ **If storing pointers, always create a new local variable before storing.**

---

### **Bad Example (Loop Variable Reused, Causes Incorrect Pointers)**

```go
type Customer struct {
	ID      string
	Balance float64
}

type Store struct {
	m map[string]*Customer
}

func (s *Store) storeCustomers(customers []Customer) {
	for _, customer := range customers { // ❌ Loop variable is reused across iterations
		s.m[customer.ID] = &customer // ❌ All map values point to the last element
	}
}

func main() {
	s := Store{m: make(map[string]*Customer)}
	s.storeCustomers([]Customer{
		{ID: "1", Balance: 10},
		{ID: "2", Balance: -10},
		{ID: "3", Balance: 0},
	})

	for k, v := range s.m {
		fmt.Printf("key=%s, value=%+v\n", k, v) // ❌ All keys point to the last struct
	}
}
```

🔴 **Why is this wrong?**

- The **loop variable (`customer`) is reused** in each iteration, so all stored pointers **reference the last element (`ID: "3"`)**.
- **All keys in the map point to the same struct instance**, causing incorrect behavior.

---

### **Good Example #1 (Using a Local Variable to Store Unique Pointers)**

```go
func (s *Store) storeCustomers(customers []Customer) {
	for _, customer := range customers {
		current := customer // ✅ Creates a new variable in each iteration
		s.m[current.ID] = &current // ✅ Now stores unique pointers
	}
}
```

🔵 **Why is this better?**

- **Each iteration gets its own variable (`current`)**, ensuring distinct memory locations.
- **Stored pointers now correctly reference unique `Customer` instances.**

---

### **Good Example #2 (Using Index-Based Iteration to Avoid Reused Variables)**

```go
func (s *Store) storeCustomers(customers []Customer) {
	for i := range customers {
		s.m[customers[i].ID] = &customers[i] // ✅ References original slice elements
	}
}
```

🔵 **Why is this better?**

- **Uses `customers[i]` directly, avoiding loop variable reuse.**
- **More efficient than copying structs, especially for large slices.**

---

### **Key Takeaways:**

- **Range loops reuse the loop variable**, leading to unexpected pointer references.
- **Use a local variable inside the loop (`current := customer`) to store correct references.**
- **Index-based iteration (`customers[i]`) ensures unique memory references.** 🚀