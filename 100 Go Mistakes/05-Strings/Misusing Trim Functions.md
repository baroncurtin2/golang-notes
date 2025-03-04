### **Problem:**

Developers often **confuse `TrimRight` with `TrimSuffix`** (and similarly, `TrimLeft` with `TrimPrefix`), leading to **unexpected string modifications**.

---

### **Solution:**

✅ **Use `TrimRight` and `TrimLeft` when removing multiple trailing/leading characters.**  
✅ **Use `TrimSuffix` and `TrimPrefix` when removing an exact substring from the start or end.**

---

### **Best Practices:**

✅ **Use `TrimRight` / `TrimLeft` when trimming multiple occurrences of characters.**  
✅ **Use `TrimSuffix` / `TrimPrefix` when checking and removing a specific substring.**  
✅ **Use `Trim` when removing characters from both ends.**

---

### **Bad Example (Misusing `TrimRight` Instead of `TrimSuffix`)**

```go
fmt.Println(strings.TrimRight("123oxo", "xo")) // ❌ Unexpected output: "123"
```

🔴 **Why is this wrong?**

- `TrimRight("123oxo", "xo")` **removes ALL trailing `x` and `o` characters**, not just `"xo"`.
- Expected output might be `"123o"`, but it **removes additional `o` and `x` characters**.

---

### **Good Example (Using `TrimSuffix` to Remove an Exact Suffix)**

```go
fmt.Println(strings.TrimSuffix("123oxo", "xo")) // ✅ Correct output: "123o"
```

🔵 **Why is this better?**

- **Only removes the exact `"xo"` suffix**, leaving the rest of the string intact.

---

### **Bad Example (Misusing `TrimLeft` Instead of `TrimPrefix`)**

```go
fmt.Println(strings.TrimLeft("oxo123", "ox")) // ❌ Unexpected output: "123"
```

🔴 **Why is this wrong?**

- `TrimLeft("oxo123", "ox")` **removes ALL leading `o` and `x` characters**, not just `"ox"`.

---

### **Good Example (Using `TrimPrefix` to Remove an Exact Prefix)**

```go
fmt.Println(strings.TrimPrefix("oxo123", "ox")) // ✅ Correct output: "o123"
```

🔵 **Why is this better?**

- **Removes only the `"ox"` prefix**, keeping the remaining `"o123"` intact.

---

### **When to Use Each Function:**

|**Function**|**Use Case**|**Example Input**|**Output**|
|---|---|---|---|
|`TrimRight`|Remove all trailing runes in a set|`"123oxo"` with `"xo"`|`"123"`|
|`TrimLeft`|Remove all leading runes in a set|`"oxo123"` with `"ox"`|`"123"`|
|`TrimSuffix`|Remove an exact suffix if present|`"123oxo"` with `"xo"`|`"123o"`|
|`TrimPrefix`|Remove an exact prefix if present|`"oxo123"` with `"ox"`|`"o123"`|
|`Trim`|Remove leading and trailing runes in a set|`"oxo123oxo"` with `"ox"`|`"123"`|

---

### **Key Takeaways:**

- **`TrimRight` and `TrimLeft` remove multiple characters from a set** until a non-matching rune is found.
- **`TrimSuffix` and `TrimPrefix` remove only the exact substring match.**
- **Use `Trim` when removing characters from both ends of a string.** 🚀