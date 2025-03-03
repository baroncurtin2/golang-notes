### **Problem:**

Integer literals **starting with `0` are interpreted as octal (base 8)** in Go, which can lead to **unexpected calculations and confusion** in the code.

---

### **Solution:**

✅ **Use `0o` prefix for octal numbers** – Improves clarity and prevents misinterpretation.  
✅ **Be explicit with number bases** – Use `0b` for binary, `0x` for hexadecimal, and `0o` for octal.  
✅ **Use underscores (`_`) for readability** – Helps when dealing with large numbers.

---

### **Best Practices:**

✅ **Avoid leading zeros in decimal numbers** – They can be mistakenly interpreted as octal.  
✅ **Use `0o` instead of just `0` for octal literals** – Prevents confusion for readers.  
✅ **Use separators (`_`) in long numbers** – Improves readability without affecting value.

---

### **Bad Example (Confusing Octal Representation)**

```go
sum := 100 + 010  // ❌ 010 is octal (8 in decimal), not 10
fmt.Println(sum)  // Output: 108 (not 110)
```

🔴 **Why is this wrong?**

- The leading `0` makes `010` an **octal number (base 8)**, which is **8 in decimal**.
- The expected output might be `110`, but the actual result is `108`.

---

### **Good Example (Using `0o` for Clarity)**

```go
sum := 100 + 0o10  // ✅ Explicit octal representation
fmt.Println(sum)   // Output: 108
```

🔵 **Why is this better?**

- **Clearly indicates octal notation (`0o10` = 8 in decimal).**
- **Avoids confusion** when reading or debugging the code.

---

### **Good Example (Using Readable Number Formats)**

```go
file, err := os.OpenFile("foo", os.O_RDONLY, 0o644) // ✅ Uses 0o prefix for octal

number := 1_000_000_000  // ✅ Uses underscores for readability
binary := 0b1010         // ✅ Binary (10 in decimal)
hex := 0xFF              // ✅ Hexadecimal (255 in decimal)
```

🔵 **Why is this better?**

- **Improves readability** with clear number base prefixes.
- **Avoids misinterpretation of leading zeros.**

---

### **When to Use `0o` for Octal Numbers:**

✅ When defining **file permissions** (e.g., `0o644` for Linux permissions).  
✅ When working with **bit masks and flags** that require octal notation.

### **When to Avoid Octal Notation:**

❌ When using **leading zeros in decimal numbers**—they may be mistakenly interpreted as octal.  
❌ When **decimal representation is more intuitive** for general calculations.

---

### **Key Takeaways:**

- **Octal numbers in Go start with `0`, but prefer `0o` for clarity.**
- **Avoid leading zeros in decimal numbers to prevent confusion.**
- **Use `0b`, `0x`, and `0o` to indicate binary, hex, and octal values explicitly.**
- **Use underscores (`_`) for better number readability.** 🚀