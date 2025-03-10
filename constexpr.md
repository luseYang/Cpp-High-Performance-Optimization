在编译时计算表达式结果，减少运行时开销

# 减少运行时计算
`constexpr` 允许在编译时计算表达式，从而减少运行时的计算量。例如：

```cpp
constexpr int factorial(int n) {
    return (n <= 1) ? 1 : (n * factorial(n - 1));
}

constexpr int result = factorial(5); // 在编译期计算，运行时无需计算
```
# 优化分支预测
当 `constexpr`作用于 `if` 语句时，编译器可以在编译期直接优化掉无用的分支：

```cpp
constexpr bool useOptimizedPath = true;

void compute() {
    if constexpr (useOptimizedPath) { // 直接在编译期决定代码路径
        // 快速路径
    } else {
        // 备用路径
    }
}
```

`if constexpr` 允许编译器在编译期直接消除不必要的分支，提高运行效率。












































