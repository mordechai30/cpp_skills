# Cpp17

## C++17 Features

### Structured Bindings

Decompose objects into individual variables:

```cpp
// Pairs and tuples
std::pair<int, std::string> get_data() {
    return {42, "Answer"};
}

auto [id, name] = get_data();
std::cout << id << ": " << name << std::endl;

// Maps
std::map<std::string, int> scores = {{"Alice", 95}, {"Bob", 87}};
for (const auto& [name, score] : scores) {
    std::cout << name << " scored " << score << std::endl;
}

// Arrays
int arr[3] = {1, 2, 3};
auto [a, b, c] = arr;

// Structs
struct Point { int x, y; };
Point p{10, 20};
auto [x, y] = p;
```

### If Constexpr

Compile-time conditional statements:

```cpp
template<typename T>
auto process(T value) {
    if constexpr (std::is_integral_v<T>) {
        return value * 2;
    } else if constexpr (std::is_floating_point_v<T>) {
        return value * 3.14;
    } else {
        return value;
    }
}

int result1 = process(10);      // Returns 20
double result2 = process(2.0);  // Returns 6.28
```

### Fold Expressions

Variadic template operations:

```cpp
// Sum all arguments
template<typename... Args>
auto sum(Args... args) {
    return (args + ...);
}

int total = sum(1, 2, 3, 4, 5); // 15

// Print all arguments
template<typename... Args>
void print(Args... args) {
    (std::cout << ... << args) << std::endl;
}

print("Value: ", 42, " ", 3.14); // Value: 42 3.14

// Logical operations
template<typename... Args>
bool all_true(Args... args) {
    return (... && args);
}

bool result = all_true(true, true, false); // false
```

### Std::optional

Type-safe optional values:

```cpp
#include <optional>

std::optional<int> find_value(const std::vector<int>& vec, int target) {
    auto it = std::find(vec.begin(), vec.end(), target);
    if (it != vec.end()) {
        return *it;
    }
    return std::nullopt;
}

// Usage
std::vector<int> numbers = {1, 2, 3, 4, 5};
if (auto result = find_value(numbers, 3)) {
    std::cout << "Found: " << *result << std::endl;
} else {
    std::cout << "Not found" << std::endl;
}

// Value_or for default
int value = find_value(numbers, 10).value_or(-1);
```

### Std::variant

Type-safe union:

```cpp
#include <variant>

std::variant<int, double, std::string> data;

data = 42;
data = 3.14;
data = std::string("Hello");

// Visit pattern
std::visit([](auto&& arg) {
    using T = std::decay_t<decltype(arg)>;
    if constexpr (std::is_same_v<T, int>) {
        std::cout << "int: " << arg << std::endl;
    } else if constexpr (std::is_same_v<T, double>) {
        std::cout << "double: " << arg << std::endl;
    } else {
        std::cout << "string: " << arg << std::endl;
    }
}, data);

// Get value
if (auto* str = std::get_if<std::string>(&data)) {
    std::cout << *str << std::endl;
}
```

### If Statement with Initializer (C++17)

```cpp
// C++17: if statement with initializer and structured binding
std::map<int, std::string> map = {{1, "one"}, {2, "two"}};

if (auto [it, inserted] = map.insert({3, "three"}); inserted) {
    std::cout << "Inserted: " << it->second << std::endl;
}

// Enhanced static constexpr if
template<typename T>
void process(T value) {
    if constexpr (requires { value.size(); }) {
        std::cout << "Size: " << value.size() << std::endl;
    }
}
```
