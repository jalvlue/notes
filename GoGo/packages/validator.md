在 `github.com/go-playground/validator` 包中，有一些内置的验证器和标签可用于对字段进行验证。以下是一些常用的验证方式：

1. `required`：确保字段的值不为空。对应的标签为 `required`。
2. `min` 和 `max`：用于验证数字类型字段的最小值和最大值。标签格式为 `min:值` 和 `max:值`。
3. `len`：用于验证字符串、数组、切片和映射字段的长度。标签格式为 `len:值`。
4. `email`：验证字段是否为有效的电子邮件地址。对应的标签为 `email`。
5. `url`：验证字段是否为有效的 URL。对应的标签为 `url`。
6. `alpha`：验证字段是否只包含字母字符。对应的标签为 `alpha`。
7. `numeric`：验证字段是否只包含数字字符。对应的标签为 `numeric`。
8. `alphaunicode`：验证字段是否只包含 Unicode 字母字符。对应的标签为 `alphaunicode`。
9. `alphanum`：验证字段是否只包含字母和数字字符。对应的标签为 `alphanum`。
10. `alphanumunicode`：验证字段是否只包含 Unicode 字母和数字字符。对应的标签为 `alphanumunicode`。
11. `contains`：验证字段是否包含指定的子字符串。标签格式为 `contains:子字符串`。
12. `startswith` 和 `endswith`：验证字段是否以指定的前缀或后缀开头/结尾。标签格式为 `startswith:前缀` 和 `endswith:后缀`。
13. `regex`：使用正则表达式验证字段的值。标签格式为 `regex:正则表达式`。

这些只是一部分可用的验证方式，`github.com/go-playground/validator` 还提供了其他更多验证器和标签，可以根据具体需求进行查阅和使用。