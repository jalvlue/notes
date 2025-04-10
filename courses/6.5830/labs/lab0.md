# task 1 Start an http server and handle requests
- 开启一个 http 服务器来响应请求
	- 其实只有一个页面, 将课程代码框架写好的 `handlers.HomeHandler` 注册到 `/` 路径即可
- 完成 handler 的逻辑
	- 从请求中获得查询的线参数
	- 打开数据库, 根据线参数将数据查询出来 (数据类型为 `[]int`)
	- 调用实用函数绘图 (数据类型为 `[]byte`)
	- 将 `[]byte` 转换为 `base64 string`, 然后嵌入到 html 模板中
	- 将模板写回响应

![[Pasted image 20240606213838.png]]

# task 2 Run a query over a CSV file
- 直接从原生 CSV 文件中读取全部记录, 然后过滤出查询的记录, 而不是像提供的 sqlite 数据库一样, 使用 `select` 语句查询结果
- 使用 `encoding/csv` 对于 CSV 文件进行处理: [[csv]]

![[Pasted image 20240606213819.png]]