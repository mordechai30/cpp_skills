# Cpp23

## C++23 Features (when supported)

### Std::expected

Error handling without exceptions:

```cpp
#include <expected>

std::expected<int, std::string> divide(int a, int b) {
    if (b == 0) {
        return std::unexpected("Division by zero");
    }
    return a / b;
}

// Usage
auto result = divide(10, 2);
if (result) {
    std::cout << "Result: " << *result << std::endl;
} else {
    std::cout << "Error: " << result.error() << std::endl;
}

// Transform and error handling
auto transformed = divide(20, 4)
    .transform([](int x) { return x * 2; })
    .or_else([](auto error) {
        std::cout << "Error: " << error << std::endl;
        return std::expected<int, std::string>(0);
    });
```
