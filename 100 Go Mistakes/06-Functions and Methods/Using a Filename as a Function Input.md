### **Problem:**

Passing a **filename** as a function input **reduces reusability and makes unit testing cumbersome**, as tests require creating multiple files.

---

### **Solution:**

✅ **Accept `io.Reader` instead of a filename** – Allows reading from files, HTTP requests, or in-memory data.  
✅ **Use `strings.NewReader()` for unit tests** – Eliminates the need for temporary files.  
✅ **Keep file handling (`os.Open`) separate from processing logic** – Improves function modularity.

---

### **Best Practices:**

✅ **Pass an `io.Reader` to abstract the data source.**  
✅ **Decouple file handling from processing logic for better testability.**  
✅ **Use `strings.NewReader()` in tests instead of creating actual files.**

---

### **Bad Example (Function Takes a Filename, Making Testing Harder)**

```go
func countEmptyLinesInFile(filename string) (int, error) {
	file, err := os.Open(filename) // ❌ Tightly coupled to file handling
	if err != nil {
		return 0, err
	}
	defer file.Close()

	scanner := bufio.NewScanner(file)
	count := 0
	for scanner.Scan() {
		if scanner.Text() == "" {
			count++
		}
	}
	return count, nil
}
```

🔴 **Why is this wrong?**

- **Hard to test** – Requires creating actual files for every test case.
- **Not reusable** – Cannot be used for HTTP requests, strings, or other input sources.

---

### **Good Example (Using `io.Reader` for Reusability & Testability)**

```go
func countEmptyLines(reader io.Reader) (int, error) { // ✅ Accepts io.Reader
	scanner := bufio.NewScanner(reader)
	count := 0
	for scanner.Scan() {
		if scanner.Text() == "" {
			count++
		}
	}
	return count, nil
}
```

🔵 **Why is this better?**

- **Abstracts the data source** – Works with files, HTTP requests, and in-memory data.
- **Easier to test** – No need for actual files.

---

### **Good Example (Decoupling File Handling from Processing Logic)**

```go
func countEmptyLinesFromFile(filename string) (int, error) {
	file, err := os.Open(filename) // ✅ Handles file separately
	if err != nil {
		return 0, err
	}
	defer file.Close()
	
	return countEmptyLines(file) // ✅ Reuses generic function
}
```

🔵 **Why is this better?**

- **Keeps `countEmptyLines()` reusable for different input sources.**
- **Maintains a separate function for file-specific logic.**

---

### **Good Example (Unit Test Without File Creation)**

```go
func TestCountEmptyLines(t *testing.T) {
	input := `foo
bar

baz
`
	reader := strings.NewReader(input) // ✅ No need to create a file
	count, err := countEmptyLines(reader)

	if err != nil {
		t.Fatalf("unexpected error: %v", err)
	}
	if count != 1 {
		t.Errorf("expected 1 empty line, got %d", count)
	}
}
```

🔵 **Why is this better?**

- **Tests are self-contained** – No external files required.
- **Easier to maintain** – Test cases are in one place, not in separate files.

---

### **When to Use Each Approach:**

|**Scenario**|**Recommended Approach**|
|---|---|
|**Reading from a file**|Use `os.Open()` and pass `*os.File` to a function|
|**Reading from an HTTP request**|Pass `request.Body (io.Reader)`|
|**Unit testing with static data**|Use `strings.NewReader()`|
|**General processing logic**|Accept `io.Reader`|

---

### **Key Takeaways:**

- **Use `io.Reader` instead of a filename to improve testability and reusability.**
- **Decouple file handling (`os.Open`) from logic processing.**
- **Use `strings.NewReader()` in tests to avoid creating files.** 🚀