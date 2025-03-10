通过模版继承实现静态多态，避免虚函数的调用开销

核心思想是让派生类继承一个以派生类自身为模版参数的基类。

# 基本用法：

```cpp
#pragma once

template<typename Derived>
class Base {
public:
	void interface() {
		static_cast<Derived*>(this)->implementation();
	}
};

class Derived : public Base<Derived> {
public:
	void implementation() {
		std::cout << "Derived implementation\n";
	}
};

int main() {
	Derived d;
	d.interface(); // 调用 Derived::implementation()
}
```

- `Base<Derived>` 是 `Derived` 的基类，但 `Base` 内部通过 `static_cast<Derived*>(this)` 访问 `Derived` 的成员。

- 这允许 `Base` 在编译时知道 `Derived` 的类型，从而提供静态多态，避免了虚函数的开销。

# 避免虚函数的开销

`CRTP` 可以在不使用虚函数的情况下实现类似多态的行为：

```cpp
#include <iostream>

template <typename Derived>
class Base {
public:
    void doWork() {
        static_cast<Derived*>(this)->work();
    }
};

class Derived1 : public Base<Derived1> {
public:
    void work() {
        std::cout << "Derived1 working\n";
    }
};

class Derived2 : public Base<Derived2> {
public:
    void work() {
        std::cout << "Derived2 working\n";
    }
};

int main() {
    Derived1 d1;
    Derived2 d2;

    d1.doWork(); // 调用 Derived1::work()
    d2.doWork(); // 调用 Derived2::work()
}
```

- **避免虚函数表（vtable）**，减少运行时开销。

- **编译期确定函数调用**，可以进行内联优化。

# 扩展基类功能

CRTP 允许基类为派生类提供额外功能，如计数对象数量。

```cpp
template <typename Derived>
class Counter {
    inline static int count = 0;
public:
    Counter() { ++count; }
    ~Counter() { --count; }

    static int getCount() { return count; }
};

class A : public Counter<A> {};
class B : public Counter<B> {};

int main() {
    A a1, a2;
    B b1, b2, b3;

    std::cout << "A count: " << A::getCount() << "\n"; // 输出 2
    std::cout << "B count: " << B::getCount() << "\n"; // 输出 3
}
```

- `Counter<T>` 维护 `T` 类型的对象计数。
- `inline static` 变量确保不同派生类的计数互不干扰。

# 增强派生类

CRTP 可用于增强派生类的功能，例如添加比较操作符：

```cpp
template <typename Derived>
class Comparable {
public:
    bool operator==(const Derived& other) const {
        return static_cast<const Derived*>(this)->isEqual(other);
    }

    bool operator!=(const Derived& other) const {
        return !(*this == other);
    }
};

class Point : public Comparable<Point> {
public:
    int x, y;
    Point(int x, int y) : x(x), y(y) {}

    bool isEqual(const Point& other) const {
        return x == other.x && y == other.y;
    }
};

int main() {
    Point p1(1, 2), p2(1, 2), p3(3, 4);

    std::cout << (p1 == p2) << "\n"; // 1
    std::cout << (p1 != p3) << "\n"; // 1
}
```

- `Comparable<T>` 使 `Point` 具备 `==` 和 `!=` 操作符，而无需手写重复代码。
利用 CRTP 模式进行静态多态，使 `isEqual` 由 `Point` 提供，实现具体逻辑。

# CRTP + SFINAE

```cpp
#include <type_traits>
#include <iostream>

template <typename Derived>
class HasToString {
public:
    std::string toString() const {
        /*
        * static_assert 静态断言，进行编译时断言检查。
        声明静态断言。如果断言失败，那么程序非良构，并且可能会生成诊断错误信息。
        获取 Derived::toString 成员函数的类型, 与 std::string(Derived::*)() const 比较（这是一个指向 Derived 类成员函数的指针类型）
        */ 
        static_assert(std::is_same_v<decltype(&Derived::toString), std::string(Derived::*)() const>,
            "Derived must implement std::string toString() const");
        return static_cast<const Derived*>(this)->toString();
    }
};

class Valid : public HasToString<Valid> {
public:
    std::string toString() const {
        return "I am Valid";
    }
};

// 错误：Invalid 没有 toString 方法
// class Invalid : public HasToString<Invalid> {};

int main() {
    Valid v;
    std::cout << v.toString() << "\n";
}
```

- 通过 static_assert 确保 Derived 必须 实现 toString，否则编译失败。

- 避免运行时错误，在编译期进行约束检查。

# CRTP 与类型擦除

CRTP 可以用于替代运行时多态（类型擦除），避免虚表：

```cpp
#include <iostream>

template <typename Derived>
class Drawable {
public:
    void draw() {
        static_cast<Derived*>(this)->drawImpl();
    }
};

class Circle : public Drawable<Circle> {
public:
    void drawImpl() { std::cout << "Drawing Circle\n"; }
};

class Square : public Drawable<Square> {
public:
    void drawImpl() { std::cout << "Drawing Square\n"; }
};

int main() {
    Circle c;
    Square s;

    c.draw(); // Drawing Circle
    s.draw(); // Drawing Square
}
```








