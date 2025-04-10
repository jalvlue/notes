# org-framework-service
- 端口在 `wsl:10999`
```json
{
  // Use IntelliSense to learn about possible attributes.
  // Hover to view descriptions of existing attributes.
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "name": "DEBUG org_framework_service",
      "type": "go",
      "request": "launch",
      "mode": "auto",
      "program": "${fileDirname}",
      "args": [
        "-e",
        "local",
        "-p",
        "10999",
        "-g",
        "stage"
      ]
    }
  ]
}
```
- `org-framework-service` 的架构非常干净, 代码结构有一点类似 `apache-answer`, 值得深入学习

### 代码架构
- `/model`, 设置 db 连接以及 ORM object 结构体
- `/repository`, 单例模式, 每一个数据库表都对应一个 `repo` 单例, `repo` 结构体中包含数据库连接指针, 同时对每一个表的 curd 方法都写成 `repo` 的方法, 后续进行 crud 就调用 `repo` 单例的方法
- `/service`, 实际业务逻辑, 向下会调用 `repo` 中的 curd 方法, 向上会被 `handler` 层调用
	- `data_transfer_object, DTO`, 在层与层之间进行数据流转的物体, 例如在 `handler` 层和 `repo` 层之间进行数据流转
- `/support`, 一些与业务无关的实用代码, 例如一些数据结构实现, 算法实现, 与操作系统交互等
- `/xhttp`, 接口层, 主要写了很多绑定到一些 URL 上的 `handler`
	- `handler`, 实际的路由处理函数, 这里用的是 `gin` 框架, 因此都是 `gin.HandlerFunc`, `handler` 中有很多 `view_object, VO`, 视图对象, 这些通常是返回给前端的临时只读对象, 同时还定义了很多 `request_object, RO`, 这些通常用来将前端的数据 (主要是 `POST` 方法的参数) unmarshal 成 RO 对象
	- `middleware`, gin 中间件, 日志, 恢复, 外部和内部 id 转换等中间间
	- `router`, 路由文件
- `main.go`
- `.env.local`


# Training-admin-go
- 端口在 `localhost:7999`
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "DEBUG training-admin-go",
      "type": "go",
      "request": "launch",
      "mode": "auto",
      "program": "${fileDirname}",
      "args": [
        "run",
        "-c",
        "../.env.local",
        "-p",
        "7999"
      ]
    }
  ]
}
```


# Training-msg-service
- 端口在 `wsl:8999`
```json
{ 
  "version": "0.2.0",
  "configurations": [
    {
      "name": "DEBUG training-msg-service",
      "type": "go",
      "request": "launch",
      "mode": "auto",
      "program": "${fileDirname}",
      "args": [
        "run",
        "-c",
        "../.env.local",
        "-p",
        "8999"
      ]
    }
  ]
}
```


# Training-script-service
- 端口在 `wsl:9999`
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "DEBUG training-script-service",
      "type": "go",
      "request": "launch",
      "mode": "auto",
      "program": "${fileDirname}",
      "args": [
        "run",
        "-c",
        "../.env.local",
        "-p",
        "9999"
      ]
    }
  ]
}
```



# drp_go
- 端口在 `wsl:10999`
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "debug drp_go",
      "type": "go",
      "request": "launch",
      "mode": "auto",
      "program": "${cwd}/cmd/main.go",
      "args": [
        "run",
        "-c",
        "${cwd}/.env.local",
        "-p",
        "10999"
      ]
    }
  ]
}
```


# 测试环境 doris
url: jdbc:mysql://172.16.6.35:9030/db_ods?autoReconnect=true&useSSL=false&characterEncoding=utf-8&zeroDateTimeBehavior=convertToNull  
          username: admin  
          password: nR@t4LJA


# 跳板机
找到对应的机器之后可以选择第三方客户端连接, 然后复制域名端口用户密码到 dg 里面连



# 日志平台
![[Pasted image 20240709100624.png]]
- 脚本(常驻任务)日志一般都在 `*-task-log` 中
- 接口日志一般都在 `*-service-log-cold` 中
- 业务日志一般都在 `*-service-log` 中
- 
