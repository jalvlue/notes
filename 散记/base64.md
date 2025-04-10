Base64 编码是一种将二进制数据转换为 ASCII 字符串的编码方式。它的编码规则和算法如下：

### Base64 编码规则

1. **字符集**：Base64 使用一个包含 64 个字符的字符集：
   ```
   A-Z (26)
   a-z (26)
   0-9 (10)
   + (1)
   / (1)
   ```

2. **分组**：
   - 将输入的二进制数据按每 3 个字节（24 位）分组。
   - 每个字节由 8 位构成，因此 3 个字节共 24 位，分为 4 组，每组 6 位。

3. **填充**：
   - 如果输入数据的字节数不是 3 的倍数，使用 `=` 字符进行填充，以确保输出的长度是 4 的倍数。
   - 例如：
     - 如果最后只剩 1 个字节，编码后会用 `==` 填充。
     - 如果剩 2 个字节，编码后会用 `=` 填充。

### Base64 编码算法步骤

1. 将输入数据转换为二进制。
2. 将二进制数据按 3 字节分组。
3. 将每组 24 位数据分为 4 组 6 位。
4. 根据 6 位的值查找 Base64 字符集，得到 4 个字符。
5. 如果需要，添加填充字符 `=`。

### Python3 实现 Base64 编码器

以下是一个简单的 Base64 编码器的实现：

```python
import base64

def base64_encode(data):
    # 将输入数据转换为字节
    if isinstance(data, str):
        data = data.encode('utf-8')  # 转换为字节

    # 使用内置的 base64 编码函数
    encoded_data = base64.b64encode(data)

    # 将编码后的字节转换为字符串
    return encoded_data.decode('utf-8')

# 示例
if __name__ == "__main__":
    original_string = "Hello, Base64!"
    encoded_string = base64_encode(original_string)
    print(f"Original: {original_string}")
    print(f"Encoded: {encoded_string}")
```

### 解释代码：

1. **导入库**：使用 Python 内置的 `base64` 模块。
2. **编码函数**：`base64_encode(data)` 函数接受一个字符串或字节序列作为输入。
   - 如果输入是字符串，就将其编码为 UTF-8 字节。
   - 使用 `base64.b64encode()` 函数进行编码。
   - 最后，将编码后的字节转换回字符串并返回。
3. **示例**：在主程序中，定义一个原始字符串，调用编码函数，并打印原始和编码后的结果。

### 输出示例
运行上述代码将输出：

```
Original: Hello, Base64!
Encoded: SGVsbG8sIEJhc2U2NCE=
```

这样，你就实现了一个简单的 Base64 编码器。
