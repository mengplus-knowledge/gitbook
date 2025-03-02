---
title: check 单元测试框架
slug: check-dan-yuan-ce-shi-kuang-jia
cover: ""
categories:
  - 嵌入式开发
tags:
  - 单元测试
halo:
  site: https://mengplus.top
  name: efaf67d8-8197-4a14-9ba4-6bc267bedf82
  publish: true
---
`Check` 是一个功能强大的 C 单元测试框架，提供了多种断言宏（Assertion Macros）来支持不同类型的测试条件判断。以下是 `Check` 支持的主要断言宏及其用途，以及如何在范围测试中使用它们。

---

### 1. **`Check` 支持的断言宏**
`Check` 提供了多种断言宏，用于检查不同的条件。以下是常用的断言宏：

#### **基本断言**
- `ck_assert(expr)`：检查表达式 `expr` 是否为真。
- `ck_assert_msg(expr, msg, ...)`：检查表达式 `expr` 是否为真，如果为假，输出自定义消息。

#### **相等性断言**
- `ck_assert_int_eq(a, b)`：检查两个整数 `a` 和 `b` 是否相等。
- `ck_assert_int_ne(a, b)`：检查两个整数 `a` 和 `b` 是否不相等。
- `ck_assert_uint_eq(a, b)`：检查两个无符号整数 `a` 和 `b` 是否相等。
- `ck_assert_float_eq(a, b, tolerance)`：检查两个浮点数 `a` 和 `b` 是否在允许的误差范围内相等。
- `ck_assert_ptr_eq(a, b)`：检查两个指针 `a` 和 `b` 是否相等。
- `ck_assert_ptr_ne(a, b)`：检查两个指针 `a` 和 `b` 是否不相等。

#### **内存断言**
- `ck_assert_mem_eq(a, b, size)`：比较两块内存 `a` 和 `b` 是否相等，大小为 `size`。
- `ck_assert_mem_ne(a, b, size)`：比较两块内存 `a` 和 `b` 是否不相等，大小为 `size`。

#### **字符串断言**
- `ck_assert_str_eq(a, b)`：检查两个字符串 `a` 和 `b` 是否相等。
- `ck_assert_str_ne(a, b)`：检查两个字符串 `a` 和 `b` 是否不相等。
- `ck_assert_str_eq_n(a, b, n)`：检查两个字符串 `a` 和 `b` 的前 `n` 个字符是否相等。

#### **范围断言**
- `ck_assert_int_ge(a, b)`：检查整数 `a` 是否大于或等于整数 `b`。
- `ck_assert_int_gt(a, b)`：检查整数 `a` 是否大于整数 `b`。
- `ck_assert_int_le(a, b)`：检查整数 `a` 是否小于或等于整数 `b`。
- `ck_assert_int_lt(a, b)`：检查整数 `a` 是否小于整数 `b`。

#### **失败断言**
- `ck_abort()`：立即终止当前测试用例，标记为失败。
- `ck_abort_msg(msg, ...)`：立即终止当前测试用例，并输出自定义消息。

---

### 2. **范围测试的实现**
范围测试的目的是验证某个值是否在预期的范围内。例如，测试某个函数的返回值是否在 `[min, max]` 之间。可以使用 `ck_assert_int_ge` 和 `ck_assert_int_le` 来实现范围测试。

#### 示例：测试返回值是否在 `[min, max]` 范围内
```c
START_TEST(test_in_range)
{
    int value = some_function(); // 假设这是被测函数
    int min = 10;
    int max = 20;

    // 检查 value 是否 >= min 且 <= max
    ck_assert_int_ge(value, min); // value >= min
    ck_assert_int_le(value, max); // value <= max
}
END_TEST
```

#### 示例：测试数组中的所有元素是否在 `[min, max]` 范围内
```c
START_TEST(test_array_in_range)
{
    int array[] = {10, 15, 20, 25};
    size_t size = sizeof(array) / sizeof(array[0]);
    int min = 10;
    int max = 25;

    for (size_t i = 0; i < size; i++)
    {
        ck_assert_int_ge(array[i], min); // array[i] >= min
        ck_assert_int_le(array[i], max); // array[i] <= max
    }
}
END_TEST
```

---

### 3. **组合使用断言**
在实际测试中，可以组合使用多个断言来验证复杂的条件。例如，测试函数的返回值是否在某个范围内，并且其指针是否非空。

#### 示例：组合测试
```c
START_TEST(test_complex_conditions)
{
    int *result = some_function(); // 假设这是被测函数

    // 检查指针是否非空
    ck_assert_ptr_nonnull(result);

    // 检查返回值是否在 [10, 20] 范围内
    ck_assert_int_ge(*result, 10); // *result >= 10
    ck_assert_int_le(*result, 20); // *result <= 20
}
END_TEST
```

---

### 4. **`Check` 的测试套件和测试用例**
`Check` 支持将多个测试用例组织到测试套件中。以下是一个完整的示例，包括范围测试。

#### 示例：完整测试套件
```c
#include <check.h>
#include <stdlib.h>

START_TEST(test_in_range)
{
    int value = 15; // 假设这是被测函数的返回值
    ck_assert_int_ge(value, 10); // value >= 10
    ck_assert_int_le(value, 20); // value <= 20
}
END_TEST

START_TEST(test_array_in_range)
{
    int array[] = {10, 15, 20, 25};
    size_t size = sizeof(array) / sizeof(array[0]);

    for (size_t i = 0; i < size; i++)
    {
        ck_assert_int_ge(array[i], 10); // array[i] >= 10
        ck_assert_int_le(array[i], 25); // array[i] <= 25
    }
}
END_TEST

Suite *range_suite(void)
{
    Suite *s;
    TCase *tc_core;

    s = suite_create("RangeTest");

    /* Core test case */
    tc_core = tcase_create("Core");

    tcase_add_test(tc_core, test_in_range);
    tcase_add_test(tc_core, test_array_in_range);

    suite_add_tcase(s, tc_core);

    return s;
}

int main(void)
{
    int number_failed;
    Suite *s;
    SRunner *sr;

    s = range_suite();
    sr = srunner_create(s);

    srunner_run_all(sr, CK_NORMAL);
    number_failed = srunner_ntests_failed(sr);
    srunner_free(sr);

    return (number_failed == 0) ? EXIT_SUCCESS : EXIT_FAILURE;
}
```

---

### 5. **总结**
- `Check` 提供了丰富的断言宏，支持条件判断、相等性检查、内存比较、范围测试等。
- 范围测试可以通过 `ck_assert_int_ge` 和 `ck_assert_int_le` 来实现。
- 可以将多个测试用例组织到一个测试套件中，使用 `srunner` 运行所有测试。

如果你有更多特定的测试需求，请告诉我，我可以进一步优化测试用例！