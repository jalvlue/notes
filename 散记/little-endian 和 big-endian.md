### Little-endian 和 Big-endian

**字节序（Byte Order）** 是指多字节数据在内存中的存储顺序。主要有两种字节序：`little-endian` 和 `big-endian`。

#### Little-endian

- **定义**：在 little-endian 中，最低有效字节（Least Significant Byte, LSB）存储在内存的最低地址，最高有效字节（Most Significant Byte, MSB）存储在最高地址。
- **示例**：对于 32 位整数 `0x12345678`，其在内存中的存储顺序为：
  ```
  地址   内容
  0x00   0x78
  0x01   0x56
  0x02   0x34
  0x03   0x12
  ```

- **常见平台**：x 86 和 x 86-64 架构（如 Intel 和 AMD 处理器）。

#### Big-endian

- **定义**：在 big-endian 中，最高有效字节（MSB）存储在内存的最低地址，最低有效字节（LSB）存储在最高地址。
- **示例**：对于同样的 32 位整数 `0x12345678`，其在内存中的存储顺序为：
  ```
  地址   内容
  0x00   0x12
  0x01   0x34
  0x02   0x56
  0x03   0x78
  ```

- **常见平台**：某些网络协议（例如 TCP/IP）和一些 RISC 架构（如 SPARC 和 PowerPC）。

### 影响与应用

- **跨平台兼容性**：在不同字节序的系统之间传输数据时，可能需要转换字节序，以确保数据的正确解析。
- **网络字节序**：网络协议通常使用 big-endian，称为网络字节序（Network Byte Order），以确保在网络传输中的一致性。

### 总结

- **Little-endian**：LSB 在低地址，常见于 x 86 系列处理器。人类可读性高
- **Big-endian**：MSB 在低地址，常见于某些网络协议和 RISC 处理器。人类可读性差

理解字节序对于进行低级编程、网络通信和数据存储至关重要。