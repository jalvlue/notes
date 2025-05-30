Bcrypt 是一种用于密码哈希函数的密码学算法。它是由 Niels Provos 和 David Mazières 设计的，旨在提供一种安全且可调节成本的密码哈希方法。

Bcrypt 的主要特点是其慢速的计算速度和可调节的成本因素。这是通过在计算哈希之前将密码输入与随机生成的盐（salt）组合，并进行多轮的迭代计算来实现的。这种设计目的是增加计算哈希所需的时间和资源，从而增加攻击者进行暴力破解或使用彩虹表等预计算攻击的成本。

Bcrypt 还提供了自动处理盐的功能，它会生成和存储随机的盐值，并将其与哈希密码一起存储。这样可以确保每个密码都使用不同的盐进行哈希，增加了密码破解的难度。

当涉及到存储用户密码或进行密码验证时，Bcrypt 是一种常见的选择。下面是一些关于 Bcrypt 的更多信息：

1. 成本因素：Bcrypt 具有可调节的成本因素，它通过增加计算哈希所需的时间和资源来提高密码破解的成本。这个成本因素称为“工作因子”（work factor），通常表示为“成本”（cost）。增加成本因素会增加计算哈希所需的时间和资源，从而增加攻击者进行暴力破解的难度。

2. 盐值（Salt）：Bcrypt 使用随机生成的盐值来增加哈希的安全性。盐值是一个随机字符串，与密码一起输入哈希函数。每个密码都使用不同的盐值进行哈希，这样即使两个用户使用相同的密码，其哈希结果也会不同。这样做可以防止使用彩虹表等预计算攻击。

3. 自动处理盐值：Bcrypt 自动处理盐值的生成和存储，无需用户手动管理盐值。在生成哈希时，Bcrypt 会自动创建随机的盐值，并将其存储在哈希结果中。在验证密码时，Bcrypt 会从哈希结果中提取盐值，并将其与输入的密码进行哈希运算，然后与存储的哈希结果进行比较。

4. 安全性：Bcrypt 被广泛认为是一种安全的密码哈希算法。其慢速的计算速度和可调节的成本因素使得暴力破解变得非常困难。即使攻击者获取到存储的哈希结果，也很难通过逆向计算得到原始密码。

5. 集成支持：Bcrypt 在许多编程语言和框架中都有集成支持。这意味着你可以使用现有的 Bcrypt 库或函数来轻松地进行密码哈希和验证操作，无需自己实现算法。

![[Pasted image 20240224211846.png]]
![[Pasted image 20240224211759.png]]

