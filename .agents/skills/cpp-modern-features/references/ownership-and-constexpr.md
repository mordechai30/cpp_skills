# Ownership And Constexpr

## Move Semantics

### Rvalue References

```cpp
class Buffer {
    char* data;
    size_t size;

public:
    // Constructor
    Buffer(size_t s) : size(s), data(new char[s]) {}

    // Copy constructor
    Buffer(const Buffer& other) : size(other.size), data(new char[size]) {
        std::copy(other.data, other.data + size, data);
    }

    // Move constructor
    Buffer(Buffer&& other) noexcept : size(other.size), data(other.data) {
        other.data = nullptr;
        other.size = 0;
    }

    // Copy assignment
    Buffer& operator=(const Buffer& other) {
        if (this != &other) {
            delete[] data;
            size = other.size;
            data = new char[size];
            std::copy(other.data, other.data + size, data);
        }
        return *this;
    }

    // Move assignment
    Buffer& operator=(Buffer&& other) noexcept {
        if (this != &other) {
            delete[] data;
            data = other.data;
            size = other.size;
            other.data = nullptr;
            other.size = 0;
        }
        return *this;
    }

    ~Buffer() { delete[] data; }
};
```

### Std::move and Perfect Forwarding

```cpp
#include <utility>

// Using std::move
std::vector<int> vec1 = {1, 2, 3, 4, 5};
std::vector<int> vec2 = std::move(vec1); // vec1 remains valid; do not assume its value

// Perfect forwarding
template<typename T>
void wrapper(T&& arg) {
    // Forward arg preserving its value category
    process(std::forward<T>(arg));
}

// Factory function with perfect forwarding
template<typename T, typename... Args>
std::unique_ptr<T> make_unique_custom(Args&&... args) {
    return std::unique_ptr<T>(new T(std::forward<Args>(args)...));
}

// Emplace methods use perfect forwarding
std::vector<std::pair<int, std::string>> vec;
vec.emplace_back(1, "one"); // Constructs in-place
```

## Lambda Advanced Features

### Recursive Lambdas

```cpp
// C++14 and later
auto fibonacci = [](int n) {
    auto impl = [](int n, auto& self) -> int {
        if (n <= 1) return n;
        return self(n-1, self) + self(n-2, self);
    };
    return impl(n, impl);
};

// C++23: Deducing this
auto fibonacci_cpp23 = [](this auto self, int n) -> int {
    if (n <= 1) return n;
    return self(n-1) + self(n-2);
};
```

### Init Captures (C++14)

```cpp
// Move-only types in captures
auto ptr = std::make_unique<int>(42);
auto lambda = [p = std::move(ptr)]() {
    std::cout << *p << std::endl;
};

// Initialize new variable in capture
auto lambda2 = [value = computeExpensiveValue()]() {
    return value * 2;
};

// Multiple init captures
auto lambda3 = [x = getValue(), y = getOtherValue()]() {
    return x + y;
};
```

## Compile-Time Evaluation

### Constexpr Functions

```cpp
// C++11 constexpr (limited)
constexpr int factorial_cpp11(int n) {
    return (n <= 1) ? 1 : n * factorial_cpp11(n - 1);
}

// C++14 constexpr (more powerful)
constexpr int factorial(int n) {
    int result = 1;
    for (int i = 2; i <= n; ++i) {
        result *= i;
    }
    return result;
}

constexpr int value = factorial(5); // Computed at compile time

// C++20 constexpr with std::vector
constexpr auto compute_primes(int max) {
    std::vector<int> primes;
    for (int i = 2; i < max; ++i) {
        bool is_prime = true;
        for (int p : primes) {
            if (i % p == 0) {
                is_prime = false;
                break;
            }
        }
        if (is_prime) primes.push_back(i);
    }
    return primes;
}
```

### Consteval (C++20)

Functions that must be evaluated at compile time:

```cpp
consteval int square(int n) {
    return n * n;
}

constexpr int value1 = square(5); // OK: compile-time
// int x = 5;
// int value2 = square(x); // ERROR: must be compile-time

// Consteval with constexpr
constexpr int maybe_compile_time(int n) {
    if (std::is_constant_evaluated()) {
        return n * n;
    } else {
        return n + n;
    }
}
```
