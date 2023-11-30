# for_each和函数对象

### 1，for_each

std::for_each 算法函数主要用于遍历  容器中的每个元素，并将每个元素传递给函数对象 `MyFunctor` 的 `operator()` 成员函数进行处理(如下例所示)

```
std::vector<int> numbers = {1, 2, 3, 4, 5};
std::for_each(numbers.begin(), numbers.end(), MyFunctor());
```

```
struct MyFunctor {
    void operator()(int value) const {
        std::cout << "Value: " << value << std::endl;
    }
};
```

这儿for_each遍历每个元素，把每个元素当作第三个函数对象MyFunctor的参数。所以以上如果Lambda表达是来写的话，就如下所示了。

```
std::vector<int> numbers = {1, 2, 3, 4, 5};
std::for_each(numbers.begin(), numbers.end(), [](int value) {
    std::cout << "Value: " << value << std::endl;
});
```

这儿(int value)这个参数定义很重要，表示for_each里面便利的每个元素。

### 2，函数对象

函数对象（Function Object）是一个类对象，它可以像函数一样被调用。函数对象重载了圆括号运算符 `()`，使得对象可以被当作函数来调用，并执行特定的操作。

函数对象可以用作算法函数的参数，例如 `std::for_each`、`std::transform`、`std::sort` 等。通过传递函数对象作为参数，算法函数可以在每个元素上执行自定义的操作或进行其他定制行为。

函数对象可以是一个普通的类，也可以是一个类的实例（对象）。在函数对象的类定义中，你需要定义一个公共的 `operator()` 成员函数，该成员函数定义了函数对象的行为。

