### **Problem:**

Handling optional configuration parameters in Go can lead to **inflexible APIs** if done incorrectly. Using multiple function parameters or configuration structs can make APIs harder to maintain, introduce breaking changes, and reduce usability.

---

### **Solution:**

✅ **Use the Functional Options Pattern** – Allows flexible configuration while keeping the API clean and backward-compatible.  
✅ **Avoid using structs with pointer fields** – Makes API usage cumbersome and harder to read.  
✅ **Leverage variadic functions** – Enable passing multiple options without requiring empty structs.

---

### **Best Practices:**

✅ **Encapsulate configuration inside an unexported struct** – Keeps internal details hidden.  
✅ **Use functions (`WithXyz()`) to define options** – Provides type safety and validation.  
✅ **Process options inside the constructor (`NewXyz()`)** – Ensures all configurations are properly applied.

---

### **Bad Example (Using a Config Struct with Pointers)**

```go
type Config struct {
    Port *int
}

func NewServer(addr string, cfg Config) { // ❌ Clients must pass an empty struct
}

port := 8080
server := NewServer("localhost", Config{Port: &port}) // ❌ Inconvenient usage
```

🔴 **Why is this wrong?**

- Clients **must manually create pointers**, making API usage awkward.
- Requires passing an **empty struct** (`Config{}`) when no options are needed.

---

### **Good Example (Using the Functional Options Pattern)**

```go
package httplib

import (
    "errors"
    "net/http"
)

type options struct {
    port *int
}

type Option func(*options) error

// ✅ Option function for setting the port
func WithPort(port int) Option {
    return func(o *options) error {
        if port < 0 {
            return errors.New("port should be positive")
        }
        o.port = &port
        return nil
    }
}

// ✅ Constructor using variadic functional options
func NewServer(addr string, opts ...Option) (*http.Server, error) {
    var options options
    for _, opt := range opts {
        if err := opt(&options); err != nil {
            return nil, err
        }
    }

    port := 8080 // Default port
    if options.port != nil {
        port = *options.port
    }

    // Server initialization logic here...
    return &http.Server{Addr: addr}, nil
}

// ✅ Client usage
server, err := httplib.NewServer("localhost", httplib.WithPort(8080))
```

🔵 **Why is this better?**

- **No unnecessary struct instantiation** – Clients don’t need to create an empty config struct.
- **Flexible and extensible** – New options can be added **without breaking existing code**.
- **Cleaner function signatures** – Only mandatory parameters are required.

---

### **When to Use Functional Options:**

✅ When an API has **optional parameters** that may expand over time.  
✅ When **backward compatibility** is important.  
✅ When improving **readability and usability** of an API.

### **When to Avoid Functional Options:**

❌ When all parameters are **mandatory**.  
❌ When the function has only **one or two options** that won’t change.

---

### **Key Takeaways:**

- **Use functional options for clean, extensible APIs.**
- **Avoid pointer-based config structs—prefer variadic options instead.**
- **Enable optional configurations without requiring empty struct parameters.**
- **Ensure APIs remain backward-compatible and easy to use.** 🚀