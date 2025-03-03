### **Problem:**

Floating-point arithmetic in Go **is not exact** due to **precision limitations**. This can lead to **unexpected results in calculations, inaccurate comparisons, and accumulation of errors in operations**.

---

### **Solution:**

✅ **Compare floating-point numbers using a tolerance (delta), not `==`.**  
✅ **Perform arithmetic operations in an order that minimizes precision loss.**  
✅ **Use `math.IsNaN` and `math.IsInf` to check for NaN and infinite values.**  
✅ **Be mindful of cumulative rounding errors in iterative calculations.**

---

### **Best Practices:**

✅ **Avoid direct equality comparisons** – Instead, check if the difference is within a small threshold.  
✅ **Perform operations in an order that reduces rounding errors** – Group numbers of similar magnitude together before adding or subtracting.  
✅ **Use `float64` over `float32` for better precision** unless memory constraints require otherwise.

---

### **Bad Example (Incorrect Floating-Point Comparison)**

```go
package main

import "fmt"

func main() {
	a := 0.1 + 0.2
	if a == 0.3 { // ❌ Direct comparison fails due to precision loss
		fmt.Println("Equal")
	} else {
		fmt.Println("Not Equal") // Output: Not Equal
	}
}
```

🔴 **Why is this wrong?**

- **Floating-point precision loss** makes `0.1 + 0.2` slightly different from `0.3`.
- **Direct equality (`==`) fails** due to tiny precision errors.

---

### **Good Example (Using a Tolerance for Comparison)**

```go
package main

import (
	"fmt"
	"math"
)

func main() {
	a := 0.1 + 0.2
	if math.Abs(a-0.3) < 1e-9 { // ✅ Checks if the difference is within a small range
		fmt.Println("Equal")
	} else {
		fmt.Println("Not Equal")
	}
}
```

🔵 **Why is this better?**

- **Uses `math.Abs()` to compare within a small margin of error (`1e-9`).**
- **Accounts for minor precision errors.**

---

### **Good Example (Handling NaN and Infinity Properly)**

```go
package main

import (
	"fmt"
	"math"
)

func main() {
	var x float64

	posInf := 1 / x    // ✅ +Inf
	negInf := -1 / x   // ✅ -Inf
	nan := x / x       // ✅ NaN

	fmt.Println(math.IsInf(posInf, 1)) // true
	fmt.Println(math.IsInf(negInf, -1)) // true
	fmt.Println(math.IsNaN(nan)) // true
}
```

🔵 **Why is this useful?**

- **Detects and handles `NaN` and infinite values correctly.**
- **Prevents undefined behavior in calculations.**

---

### **Good Example (Minimizing Floating-Point Errors in Iterations)**

```go
func sumLargeNumbersFirst(n int) float64 {
	result := 0.0
	for i := 0; i < n; i++ {
		result += 1.0001 // ❌ Adding small numbers first increases error
	}
	return result + 10_000
}

func sumLargeNumberLast(n int) float64 {
	result := 10_000.0
	for i := 0; i < n; i++ {
		result += 1.0001 // ✅ Adding smaller numbers to a large base minimizes error
	}
	return result
}
```

🔵 **Why is the second function (`sumLargeNumberLast`) better?**

- **It groups numbers of similar magnitude before adding smaller values, reducing cumulative precision loss.**

---

### **When to Be Careful with Floating-Point Arithmetic:**

✅ When comparing values, **use a small delta instead of `==`**.  
✅ When performing large calculations, **group operations to minimize precision loss**.  
✅ When handling division, **check for `NaN` or infinity using `math.IsNaN` and `math.IsInf`**.

### **When to Use `float64` Over `float32`:**

✅ When **higher precision** is required (e.g., scientific calculations, financial applications).  
✅ When **working with large numbers** where rounding errors accumulate.

---

### **Key Takeaways:**

- **Floating-point operations are approximations—avoid direct equality checks.**
- **Use `math.Abs()` for comparisons with a tolerance.**
- **Perform arithmetic in an order that reduces rounding errors.**
- **Use `math.IsNaN` and `math.IsInf` to handle special cases.** 🚀