### **Problem:**

Excessive nesting in code increases cognitive load, making it harder to build a mental model of the function’s execution flow. Deeply nested `if/else` blocks obscure the expected logic, making the code harder to read and maintain.

### **Solution:**

1. **Reduce Nesting with Early Returns** – If an `if` block results in an immediate return, avoid using `else` and instead return early.
2. **Align the Happy Path to the Left** – Keeping the primary execution flow aligned to the left improves readability by making it easy to scan.
3. **Invert Non-Happy Paths** – Flip conditional checks so that error cases return early, keeping the main logic uncluttered.