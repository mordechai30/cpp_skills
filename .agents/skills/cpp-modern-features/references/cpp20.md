# Cpp20

## C++20 Features

### Concepts

Constrain template parameters:

```cpp
#include <concepts>

// Define a concept
template<typename T>
concept Numeric = std::is_arithmetic_v<T>;

// Use concept to constrain template
template<Numeric T>
T add(T a, T b) {
    return a + b;
}

// Concept with requires clause
template<typename T>
concept Hashable = requires(T a) {
    { std::hash<T>{}(a) } -> std::convertible_to<std::size_t>;
};

// Multiple constraints
template<typename T>
concept Sortable = std::copyable<T> && requires(T a, T b) {
    { a < b } -> std::convertible_to<bool>;
};

// Use in function
template<Sortable T>
void sort_data(std::vector<T>& vec) {
    std::sort(vec.begin(), vec.end());
}
```

### Ranges

Composable algorithms and views:

```cpp
#include <ranges>

std::vector<int> numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

// Filter and transform using views
auto result = numbers
    | std::views::filter([](int n) { return n % 2 == 0; })
    | std::views::transform([](int n) { return n * n; });

for (int n : result) {
    std::cout << n << " "; // 4 16 36 64 100
}

// Take first N elements
auto first_five = numbers | std::views::take(5);

// Reverse view
auto reversed = numbers | std::views::reverse;

// Chaining multiple operations
auto complex_view = numbers
    | std::views::filter([](int n) { return n > 3; })
    | std::views::transform([](int n) { return n * 2; })
    | std::views::take(3);
```

### Coroutines

Cooperative multitasking:

```cpp
#include <coroutine>
#include <iostream>

// Simple generator
struct Generator {
    struct promise_type {
        int current_value;

        Generator get_return_object() {
            return Generator{std::coroutine_handle<promise_type>::from_promise(*this)};
        }

        std::suspend_always initial_suspend() { return {}; }
        std::suspend_always final_suspend() noexcept { return {}; }
        std::suspend_always yield_value(int value) {
            current_value = value;
            return {};
        }

        void return_void() {}
        void unhandled_exception() {}
    };

    std::coroutine_handle<promise_type> handle;

    Generator(std::coroutine_handle<promise_type> h) : handle(h) {}
    ~Generator() { if (handle) handle.destroy(); }

    int value() { return handle.promise().current_value; }
    bool next() {
        handle.resume();
        return !handle.done();
    }
};

Generator counter(int start, int end) {
    for (int i = start; i < end; ++i) {
        co_yield i;
    }
}

// Usage
auto gen = counter(0, 5);
while (gen.next()) {
    std::cout << gen.value() << " ";
}
```

### Three-Way Comparison (Spaceship Operator)

Simplified comparison operators:

```cpp
#include <compare>

struct Point {
    int x, y;

    // Single operator generates all six comparison operators
    auto operator<=>(const Point& other) const = default;
};

Point p1{1, 2};
Point p2{1, 3};

bool eq = (p1 == p2);  // false
bool lt = (p1 < p2);   // true
bool gt = (p1 > p2);   // false

// Custom spaceship operator
struct Person {
    std::string name;
    int age;

    auto operator<=>(const Person& other) const {
        if (auto cmp = name <=> other.name; cmp != 0)
            return cmp;
        return age <=> other.age;
    }
};
```
