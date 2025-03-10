编译器的优化技术，消除函数返回值的临时对象拷贝

在 C++ 中，RVO（返回值优化） 和 NRVO（命名返回值优化） 是编译器用于消除临时对象拷贝的优化技术。它们通过直接在目标位置构造返回值，避免额外的拷贝或移动操作，从而显著提升性能。

# RVO

定义：当函数返回一个匿名临时对象时，编译器直接在调用则会的内存位置构造该对象，跳过拷贝/移动操作。

触发：

```cpp
std::string createString(){
	return std::string("Hello");
}
```

优化效果：避免一次拷贝构造函数调用（C++11 前）或移动构造函数调用（C++11 后）。

# NRVO

定义：当函数返回一个具名局部对象时，编译器直接在调用者的内存位置构造该对象，跳过拷贝/移动操作。

触发:

```cpp
std::string createString() {
    std::string s = "Hello";
    return s; // NRVO 可能触发（取决于编译器）
}
```

优化效果：与 RVO 类似，但优化对象是具名的。

# 底层原理

1. 传统拷贝过程：

```cpp
std::string func() {
    std::string s = "Hello";
    return s; // 传统方式：调用拷贝构造函数生成临时对象
}

// 调用方：
std::string result = func(); // 可能触发两次拷贝（临时对象到 result）
```

2. RVO/NRVO优化过程：

- 编译器将**调用方预留的内存地址**直接传递给函数。

- 函数内部直接在预留地址构造返回值，避免中间临时对象。

```cpp
// 伪代码示例（编译器视角）：
void func(void* __hidden__result) {
    new (__hidden__result) std::string("Hello"); // 直接在目标地址构造
}

// 调用方：
std::string result;          // 预留内存
func(&result);               // 直接构造在 result 中
```

# 经典实例

1. RVO

```cpp
// 直接返回临时对象，RVO 100% 生效（C++17 强制）
std::vector<int> getVector(){
	return std::vector<int>{1, 2, 3};
}

auto vec = getVector();	// 无拷贝
```

2. NRVO

```cpp
// 返回具名对象，NRVO 可能触发（依赖编译器）
std::string getName() {
    std::string name = "Alice";
    return name; // NRVO 优化
}

auto name = getName(); // 无拷贝（若 NRVO 生效）
```

3. 无法触发NRVO的场景：

```cpp
// 多返回路径或对象被修改，NRVO 可能失效
std::string getName(bool flag) {
    std::string a = "Alice";
    std::string b = "Bob";
    if (flag) return a;
    else return b; // 无法确定返回哪个具名对象，NRVO 失效
}
```

> C++17 对 RVO 的强制要求
> 在 C++17 中，纯右值（prvalue） 的返回必须触发 RVO，称为 "guaranteed copy elision"。
```cpp
// C++17 起，RVO 强制生效
std::string s = std::string("Hello"); // 直接构造，无临时对象
```


























