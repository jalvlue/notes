# 0. 基本设计
首先是银行的数据表的设计
![[Pasted image 20231212132709.png]]
主要分为`accounts`, `transfers`, 每一次存取、转账记录表 `entries`

账户可以自己存取, 也可以通过转账转给其他账户,
以上操作都会在 `entries` 表生成新的记录

```sql
CREATE TABLE "accounts" (

  "id" bigserial PRIMARY KEY,

  "owner" varchar NOT NULL,

  "balance" bigint NOT NULL,

  "currency" varchar NOT NULL,

  "created_at" timestamptz NOT NULL DEFAULT (now())

);

  

CREATE TABLE "entries" (

  "id" bigserial PRIMARY KEY,

  "account_id" bigint NOT NULL,

  "amount" bigint NOT NULL,

  "created_at" timestamptz NOT NULL DEFAULT (now())

);

  

CREATE TABLE "transfers" (

  "id" bigserial PRIMARY KEY,

  "from_account_id" bigint NOT NULL,

  "to_account_id" bigint NOT NULL,

  "amount" bigint NOT NULL,

  "created_at" timestamptz NOT NULL DEFAULT (now())

);

  

CREATE INDEX ON "accounts" ("owner");

  

CREATE INDEX ON "entries" ("account_id");

  

CREATE INDEX ON "transfers" ("from_account_id");

  

CREATE INDEX ON "transfers" ("to_account_id");

  

CREATE INDEX ON "transfers" ("from_account_id", "to_account_id");

  

COMMENT ON COLUMN "entries"."amount" IS 'can be negative or positive';

  

COMMENT ON COLUMN "transfers"."amount" IS 'must be positive';

  

ALTER TABLE "entries" ADD FOREIGN KEY ("account_id") REFERENCES "accounts" ("id");

  

ALTER TABLE "transfers" ADD FOREIGN KEY ("from_account_id") REFERENCES "accounts" ("id");

  

ALTER TABLE "transfers" ADD FOREIGN KEY ("to_account_id") REFERENCES "accounts" ("id");
```



# 1. db migration
使用了一个小 bin: `migrate` 进行数据库版本的自动上迁和下迁,
migrate 需要按文件名顺序定义迁移的文件
比如 `000001_init_schema.up.dql`, 然后它可以读取数据库的版本,
如果是上迁, 就执行这个数据库版本后的一个 sql 脚本
下迁则执行当前版本的 down sql 脚本
一次只涉及一个版本

这里的第一个版本 up 只是简单地增加上面的表以及约束,
down 则是删除添加的表
![[Pasted image 20231212172341.png]]
![[Pasted image 20231212172350.png]]

使用 `makefile` 快速执行指令, 加入到 `.PHONY` 的对象被称为伪指令, `make` 只会帮你执行定义的指令
![[Pasted image 20231212172459.png]]

![[Pasted image 20240219225444.png]]


# 2. 使用 sqlc 快速生成 CURD 接口
`sqlc` 也是一个小 bin, 通过定义 `sqlc.ymal`, 编写好数据库 CURD 语句, sqlc 就可以自动地生成对应的 Golang CURD 接口
![[Pasted image 20240220002255.png]]

sqlc 的执行后的结果分为三部分:
1. `models.go`: 数据库表的 `golang struct` 表示
2. `db.go`: sqlc 为数据库连接查询提供的接口
3. `*.sql.go`: sql 语句对应的 golang 实现

###### sql.go
![[Pasted image 20231212173437.png]]
在 `sqlc.ymal` 中制定好出入路径, 然后在路径里写这样的 sql query,
语句上一行是给 sqlc 指定生成 golang 函数名, 以及返回值
`:one` 返回一个对象, `:many` 返回对象切片, `:exec` 不返回

编写好后, 执行 `sqlc generate` 就可以在出路径找到自动生成的函数了
account CURD:
![[Pasted image 20231212174259.png]]

本质上, 这些方法都是把原生的 sql, 以及参数丢给数据库去执行了,
然后执行后, 拿回数据库给的结果, 再把结果换成 golang 的结构, 然后返回

给 entries 和 transfers 也加上 CURD:
![[Pasted image 20231212180639.png]]
![[Pasted image 20231212180646.png]]
生成的代码跟 account 大差不差

###### models.go
除了特定的, 自己编写的查询 sql 外(比如上面的只关于 account 的 sql), sqlc 还会读取 schema 中的内容, 生成一个 `models.go` , 为每一个表生成一个对应的 `golang struct`, 
Model 还用反引号定义了 json 标签 (就是实际数据库中的列名), 在进行序列化和反序列化的时候 (json. Marshal, json. Unmarshal)可以从这些标签中得到 json 键名
![[Pasted image 20240220002031.png]]

###### db.go
并且生成一个 `Queties` 结构, 供连接数据库并进行查询
![[Pasted image 20231212174210.png]]
DBTX 这个接口有点说法, 
ExecContext 对应 `:exec`, 只执行, 不需要返回
例如:
![[Pasted image 20231212180028.png]]

QueryRowContext 对应 `:one`, 获得行结果
![[Pasted image 20231212180147.png]]

QueryContext 对应 `:many`, 获得许多行
![[Pasted image 20231212180236.png]]



# 3. db CURD unit test
golang 中, 通常把 api 与测试代码放在同一个文件夹, 命名则为 `apiFileName_test.go`
测试的过程中, 可以使用 [require](https://pkg.go.dev/github.com/stretchr/testify/require) 包方便地对测试程序的运行结果进行检查, require 用法比较简单(就类似 assert ), 过程通常是这样的:
[[require]]
![[Pasted image 20231212181747.png]]

golang 测试以包为最大单位, 一个包中有一个特殊的函数 `TestMain(m *testing.M)`, 这是包内测试的入口, 所有测试函数在执行之前都会先执行这个函数, 因此这个函数一般起到一些预定义变量的作用, 比如创建数据库连接之类的, 供测试函数使用
![[Pasted image 20231212183315.png]]

为了测试数据的方便生成, 编写一组随机生成数据的函数, 放在 `util` 包中
`init()` 每一个包都可以有数个, 用于程序开始时进行初始化设置,
程序开始后就自动被调用
![[Pasted image 20231212181921.png]]
[[rand]]
[[strings]]


类似地, entry 和 transfer 也有类似的测试接口
transfer_test:
![[Pasted image 20231213200325.png]]
entry_test:
![[Pasted image 20231213200230.png]]


# 4. transfer Transaction
转账涉及从出户减钱, 从入户加钱, 并且还要创建两个 entry,
而且其中不能出现中断, 因此放在事务中执行比较合适, 这里添加了一个转账事务
[[事务的ACID]]

事务与普通的查询使用同一个数据库连接, 但是为了将他们区分开, 让事务和普通查询的逻辑更清晰, 新引入一个 `store` 结构
这里的 `store` 使用了组合, `*Queries` 是一个默认参数, 因此 `store` 拥有了所有 `Queries` 的方法
![[Pasted image 20231213193723.png]]
可以看出, `db` 与 `Queries` 实际上都使用了同一个数据库连接
这样区分只是让 `db` 做事务, 而让 `Queries` 做普通查询, 更清晰

这一个 execTx 方法很重要
![[Pasted image 20231213194200.png]]
使用 store 中用来做事务的 db 新开了一个事务 tx, 然后用 tx 创建一个 Queries, 在事务中对数据库进行操作, 并调用传入的函数
执行具体要做的很多步骤, 例如转账就涉及: 减出户, 加入户, 给两个户加 entry, 再在 transfer 表中加上一个记录,
使用事务包裹起来, 就避免了中断的问题了

transferTransaction:
![[Pasted image 20231213194626.png]]
![[Pasted image 20231213194635.png]]

这里为了实现, 给 account 新加了一个 AddAccountBalance 的 sql, 以及生成的对应的 golang 接口
![[Pasted image 20231213195151.png]]
==TODO: 看视频回忆这个 getaccountforupdate 有啥问题, 忘关了都==


transferTransaction 也要测试:
![[Pasted image 20231213195707.png]]

# 5. avoid deadlock
![[Pasted image 20240221213101.png]]
在数据库表设计中, `entries, transfers` 两个表都引用了 `accounts.id`
在上面初始版本中, `getAccountForUpdate` 可能会修改 `accounts.id`, 为了保持约束, 因此在事务中也会获取 `entries, transfers` 的锁, 从而导致死锁

![[Pasted image 20240221213540.png]]
改成 `FOR NO KEY UPDATE`, 避免获取事务锁


# 6. use GitHub CI workflow
![[Pasted image 20240221231932.png]]
[[CI&CD--Github Actions]]


# 7. add http api
到了这一步, 就可以编写 main 开始系统的运转了
![[Pasted image 20231213201310.png]]
main 中主要做了三件事: 连接 db, 创建 store 操控 db, 创建 server 开始运行监听外来的请求

server:
![[Pasted image 20231213201748.png]]
`server` 包含了一个 `store` 用于操作数据库, 一个 `gin Engine` 用于新建路由监听请求, 这里只定义了一些关于 account 的 api 操作, 包括一个 POST 新建用户, 一个动态 GET 根据 id 获得用户信息, 一个获得全部用户的GET

对于 account, 写了几个 api(也就是对用 URL 的 handler ), 通过 http 访问 api 来操作数据库

POST createAccount 例子:
![[Pasted image 20231213203649.png]]
这里使用了带有 json tag 的一个结构, 然后使用 ctx.ShouldBindJSON 将发送进来的请求的请求体里面的 json 数据根据结构体里绑定的 tag 读到结构中, 将请求 json 数据转为 golang 数据
然后就是调用数据库接口, 对数据库进行操作
getAccount, listAccount 这些 api 也都类似

[[gin#API Examples]]

# 8. 使用 viper 导入环境变量
上面的 main 中, 一些变量比如 `db_driver`, `db_source`, `server_address` 都用常量的方式直接硬编码定义在 main 中,
这不好
因此新建 `.env` 文件, 将环境变量都放在里面, 
![[Pasted image 20240223202110.png]]

并使用 viper 包
从 `.env` 中读出, 并放到新建的系统配置 `Config` 结构中
![[Pasted image 20231213204814.png]]
其实 config 也就是把环境变量的定义放进去, 然后用 viper 从 `.env` 中读取
这里使用了 `mapstructure` 标签

main:
![[Pasted image 20231213204910.png]]
将 config 作为一个变量放在 main 函数中, 然后使用 config 读取出来的那些东西

# 9. 使用 mock 写测试
[[mock]]
[[testing]]
mock 本质上就是将原本的数据库操作变为虚拟的, 定义好行为的操作
从而提高了数据的隔离性


对于之前的 http server 响应流程:
发出请求 -- 接收请求, 转到对应的 handler -- 通过 store 访问数据库 -- 返回响应

这里的 mock 就是将原来的实际数据库 `SQLStore` 替换为 `mockStore`,
![[Pasted image 20240223221816.png]]
而 `mockStore` 通过打桩 `stubs` 定义好数据库行为, 后续通过 store 访问时验证定义好的行为, 从而跳过数据库, 只对 http api 进行测试

打桩和列表式测试用例:
![[Pasted image 20240223222948.png]]

运行测试:
使用 `t.Run(tc.name, func...)` 可以启动子测试, 让每一个测试用例都单独成为一个自测试, 更好排查
![[Pasted image 20240223223019.png]]
[[httptest]]

同时, gin 默认为 DebugMode, 会输出很多调试信息, 
因此为了让测试结果更清爽, 可以在 http api 的包中, 新增一个 TestMain 函数将 gin 设置为 TestMode
![[Pasted image 20240223224044.png]]

同样的对 `createAccountAPI, listAccountAPI` 进行测试
![[Pasted image 20240223230923.png]]
这里对 `request` 的处理可以学习一下, 相当于测试函数即当客户端又当服务端
[[net.http]]


# 10. 自定义 validator

之前提到过在结构体字段的标签中使用 `binding:validator` 进行验证绑定, 用到过 `validator` 包提供的 `required, oneof` 等
但是除此之位还可以使用自定义的验证规则
![[Pasted image 20240223232508.png]]
在 validator 包中, 验证规则都是 `validator.Func`, 将需要的规则保存到一个变量中, 然后就可以在 `binding.Validator.Engine()` 中通过该变量注册该规则, 由于 gin 使用了 `validator` 包作为验证包, 因此 http 请求将会参数将会受到这个规则的验证

![[Pasted image 20240223233327.png]]
这里将这个规则注册成了 `currency`, 因此后续可以在结构体字段中使用 `binding:currency` 进行货币种类约束

# 11. 为系统新增 user 表
![[Pasted image 20240223234820.png]]

![[Pasted image 20240223235508.png]]

为 user 表编写几个查询语句(数据库操作), 并用 sqlc 生成对应的 golang api
![[Pasted image 20240224205345.png]]


# 12. 使用 Bcrypt 安全地存储哈希密码
[[Bcrypt]] 是一种哈希算法, 
对于用户来说, 创建用户时, 不应该直接存储明文密码, 而需要哈希后的密码

用户注册 -- 初始密码 -- 使用 `Bcrypt` 进行哈希并存储在数据库中 -- 用户登录 -- 输入密码 -- 使用 `Bcrypt` 计算并与数据库中的正确值进行比较 -- 授权或不授权

golang.org 提供了一个 `bcrypt` 包,
简单而言就是提供了一个黑魔法: 数据库中只存储哈希后的密码, 根据用户登陆时提供的原始密码和数据库中的哈希后密码可以判断是否能对应起来, 对于同一个原始密码, 使用随机盐保证不生成相同的哈希密码
![[Pasted image 20240224230212.png]]
简单调用 `bcrypt.GenerateFromPassword, bcrypt.CompareHashAndPassword`, 就可以方便地生成和验证哈希密码

同时顺便在 http api 中新增一个创建用户的
![[Pasted image 20240224230546.png]]

在 server 中新增一个路由
![[Pasted image 20240224231009.png]]

![[Pasted image 20240226131626.png]]


# 13. 自定义 gomock matcher 从而编写更强的单元测试
在上面提到过, `bcrypt` 使用随机盐, 使得同一个密码多次哈希后每次都值都不同, 
[[mock#参数检测]]
因此在 gomock 中, 使用 `gomock.Eq()` 匹配器预设好的参数与实际参数在哈希密码上总是不同, 导致原本应该成功的测试失败

`gomock` 中, 所有的匹配器都实现了一个接口,
所以自定义一个 matcher, 实现该接口
![[Pasted image 20240224233419.png]]
保存了原始密码信息, 原始密码使用 `checkPassword` 对的上之后,
再使用 `reflect.DeepEqual` 进行深度比较(这里其实是模仿了 `gomock.Eq()` 中的做法)

# 14. 使用 JWT 和 PASETO 身份令牌

![[Pasted image 20231212163339.png]]
现在的应用大多是通过给用户一个有效时长有限的 `token`
然后用户后续访问时带上这个 `token`, 服务器识别有效后, 返回对应身份的服务, `token` 有效期间, 可以使用这个 `token` 多次访问服务器
[[token]]
token 其实也相当于由服务器签发, 只有服务器能验证是否有效的黑魔法
### Json Web Token-JWT
![[Pasted image 20231212163658.png]]
jwt 是 base64 明文的, 分为了三个部分: header, payload, verify signature

jwt 提供了两种签名算法:
1. 对称加密 (唯一私钥加密), 服务器拥有私有的密钥
2. 非对称加密 (公钥私钥加密), 服务器用户各有保密私钥, 私钥签名, 公钥验证
![[Pasted image 20240225002207.png]]

### Paseto
jwt 提供了太多灵活性, 因此也有一些漏洞
paseto 只提供一些标准, 更简单, 更容易使用
![[Pasted image 20240226101229.png]]
![[Pasted image 20240226101513.png]]


# 15. 实现 jwt 和 paseto token maker
对于 token maker, 需要提供生成和验证 token 的功能 `CreateToken, VerifyToken`, 为了满足 jwt 和 paseto 的多态, 这里声明成了一个接口
![[Pasted image 20231212164458.png]]

token payload:
![[Pasted image 20240226105121.png]]
[[GoGo/packages/uuid|uuid]]
payload 是一个 token 中实际存储数据的地方, 这里目前包括了四种负载数据

### 实现 jwt token maker
[[jwt-go]]
![[Pasted image 20240226105657.png]]
这里就是让 `JWTMaker` 实现了 `Maker` 接口, 注意到这里的结构只包含了密钥
然后使用 `jwt-go` 包方便地根据密钥和设置的负载生成 JWT


### 实现 paseto token maker
[[go-paseto]]
![[Pasted image 20240226114250.png]]
![[Pasted image 20240226114300.png]]

paseto 提供了较少的灵活度, 可调用的 `paseto-go` api 较少, 实现起来更加简单
直接使用 `paseto.Encrypt, paseto.Decrypt` 就可以进行新建和验证 token


# 16.用户登录 api
![[Pasted image 20231212164803.png]]
[[token]]
为了避免让用户多次提供登录凭证(账号, 密码)
可以在用户开始登录时提供一次凭证, 验证通过后, 服务器生成一个有限时长的 token, 并签名返回给用户 -- `CreateToken`
在接下来的一段有效时间内, 用户可以通过将这个 token 放到请求头中进行验证, 
服务器每次收到请求后, 将 token 进行验证, 然后提供对应服务 -- `VerifyToken`


上面提到在本地单独服务器中使用的是对称私钥加密, 在 `app.env` 中新增一个密钥变量, 再读入到 `config` 中
![[Pasted image 20240226115653.png]]

修改 server 的结构, 增加 `config, tokenMaker` 成员
![[Pasted image 20240226131830.png]]

为 user login 新增 api
用户登录就涉及到给用户生成并返回 token, 以便后续用户登陆时带上 token 来
![[Pasted image 20240226132024.png]]
![[Pasted image 20240226131422.png]]



# 17. 新增 auth 中间件
在前面, 还未给 api 增加 auth, 还未让登录生成的 token 发挥作用
现在新增 `auth` 中间件, 让持有合法 token 的请求才会被转发执行
[[gin#中间件]]
![[Pasted image 20240226162235.png]]
![[Pasted image 20240226162249.png]]


![[Pasted image 20240226164811.png]]
这里的 `auth` 中间件做的事情就是检测请求中是否带上了 `token`
如果有, 并且是合法的, 那么就将 token 中的 `payload` 解码出来, 并使用 `ctx.Set(key, value)` 存储在 `handler context` 中, 方便后续的使用
[[simplebank-BackendMasterClass#15. 实现 jwt 和 paseto token maker]]

为 auth 中间件编写测试:
![[Pasted image 20240227141127.png]]
![[Pasted image 20240227141137.png]]
测试过程实质也跟之前一样, 在测试中创建 request, 只是针对 auth 测试, 在 request 中嵌入 `authorization` 请求头, 以及 token



![[Pasted image 20240226172708.png]]
修改 server, 让 `accounts, transfers api` 用上 `auth` 中间件


修改 `account.go, transfer.go` 让其中的 api 都受到 `auth` 的限制,
对于之前每一个 api, 他们对 auth 的依赖都不同, 因此依次修改
![[Pasted image 20240226173258.png]]

[[gin#在上下文中传递值]]
使用 `ctx.MustGet(contextKey)` 获得在 auth 中间件登记的值(也就是 payload)

1. Create account: 不再需要 `owner` 请求参数, 而是从 token 中获得
![[Pasted image 20240227140418.png]]

2. Get Account: 使用 accountID 将账户查询出来后, 再在 owner 上与 token 中的信息对比, 只有属于这个用户的才返回, 否则返回 
![[Pasted image 20240226172815.png]]

3. List account: 修改 sql 语句, 新增一个 where 子句
![[Pasted image 20240227140649.png]]

4. Create transfer: 只有 from_account 属于 token 中的用户才继续
![[Pasted image 20240227140749.png]]



# 18. 在新的分支中实现新功能
[[Git]]
一般来说, 不应该将代码直接提交到 `master` 分支中, 而是新开分支, 在实现、测试完善后, 再合并到 `master` 中
![[Pasted image 20240227142223.png]]

这里新开后, 做出修改后, 可以直接 `commit + push`, 然后到 GitHub web UI 中, 新建 PR, 然后审核合并 PR

本地分支可以一直留着, 主分支则通过 `pull` 跟 GitHub 同步


# 19. 在 docker 中运行 server
之前只是 `postgresql` 数据库跑在 docker 中, 现在编写 dockerfile, 把 server 也跑在 docker 中
![[Pasted image 20240227143126.png]]
这里使用了多阶段构建 `AS` 关键字可以区分阶段
这样最终生成的镜像只包含编译出的二进制可执行程序, 非常小, 而不包括完整的 golang 编译工具链以及代码

`docker build -t simplebank:latest .`
![[Pasted image 20240227143532.png]]

### 镜像之间网络通信
现在数据库和服务器跑在不同的两个容器中, 
服务器无法通过配置中的 `localhost` 连接到数据库了,
此时可以利用 docker network 进行容器间网络通信
[[Docker#docker network]]

使用 `docker network create bank-network` 新建一个用于本应用的网络
![[Pasted image 20240227151347.png]]

使用 `docker network connect bank-network postgres12` 将数据库容器连接到这个网络
`docker network inspect bank-network`
![[Pasted image 20240227151552.png]]
可以看到这时服务器和数据库都连接到这个网络了

第一次实例化服务器容器: 
![[Pasted image 20240227151632.png]]
这里使用 `docker -e` 选项, 修改了好几个环境变量

这样就把运行在不同容器中的服务链接起来了


# 20. 使用 docker-compose 一键启动多个服务
[[Docker#docker-compose]]

![[Pasted image 20240227162526.png]]
[[Docker#docker-compose.yaml 例子]]

![[Pasted image 20240227163010.png]]

运行 `docker compose up`, 开始构建并运行
![[Pasted image 20240227155218.png]]

Docker-compose 的好处就是会自动生成一个内部网络,
因此容器之间可以直接通过服务名进行通信
![[Pasted image 20240227155138.png]]



# 21. 将应用部署到 AWS 上
TODOTODOTODO


# 22. 使用 session 保持长时状态

之前的 token 是无状态的, 不保存在数据库, 一旦泄露会比较危险, 
因此 token 的过期时间应该不能太久(10~15min)

但是这样每次 token 过期就让用户重新登录比较不好,
因此引入 `session`, 保持长时状态

![[Pasted image 20240228105241.png]]

新增 `sessions` 表, 用 sqlc 生成对应的 golang CRUD 接口
![[Pasted image 20240228141736.png]]
![[Pasted image 20240228141753.png]]

![[Pasted image 20240228141808.png]]

`sessions` 表最大的意义就是将 `refreshToken` 保持在了数据库中, 作为一种较长时间的登陆凭证, 
并且允许让用户在 `refreshToken` 还有效, `accessToken` 无效的情况下, 申请新的 `accessToken`, 避免了每次都要通过登录获得 `accessToken` 的问题

1. 修改用户登录逻辑, 不仅返回 `accessToken` 供短期使用, 还返回 `refreshToken` 供较长时间内获取新的 `accessToken`
![[Pasted image 20240228160958.png]]

2. 新增 `renewAccessToken` 方法, 允许用户通过有效的 `refreshToken` 来获得新的 `accessToken`
![[Pasted image 20240228161452.png]]

这里为了获得一些信息, 修改 `tokenMaker` 将 `payload` 返回
![[Pasted image 20240228161608.png]]


在 server 中新增一个路由
![[Pasted image 20240228161642.png]]

新增环境变量
![[Pasted image 20240228161710.png]]

疑问: `accessToken` 和 `refreshToken` 本质上并无区别, 用户甚至可以拿着 `refreshToken` 当成 `accessToken` 来用, 通过这个长时有效的来访问 api, 这样不是反而造成了泄露吗
(也许可以在后端对 api 进行处理, 或者前端区分好这两个 token )

# 24. dbdocs
TODO
P38



# 25. gRPC api
[[net.rpc]]
[[散记/gRPC|gRPC]]
使用 gin 开发 HTTP JSON api 很简单, 但是性能比使用 PRC 开发的 api 低, 因此使用 gRPC 开发 RPC api

![[Pasted image 20240228183821.png]]
![[Pasted image 20240228184240.png]]
除了 Unary RPC，还有以下几种 RPC 模式：

1. Server Streaming RPC（服务器流式 RPC）：客户端向服务器发送一个请求，服务器返回一个流式的响应。客户端可以通过流式方式获取服务器返回的多个结果。这种模式适用于客户端需要处理大量数据或持续接收流式数据的场景，如实时数据推送、日志流处理等。

2. Client Streaming RPC（客户端流式 RPC）：客户端通过流式方式发送多个请求给服务器，服务器返回一个单一的响应。客户端可以不断发送请求，并在最后接收到服务器的响应。这种模式适用于客户端需要连续发送多个数据或批量操作的场景，如上传文件、批量处理等。

3. Bidirectional Streaming RPC（双向流式 RPC）：客户端和服务器之间建立一个双向的流，双方可以通过流式方式发送多个请求和响应。客户端和服务器可以同时进行读取和写入操作，并且可以根据需要发送和接收多个消息。这种模式适用于需要实现双向通信、交互式的场景，如聊天应用、实时协作等。

这些 RPC 模式可以根据具体的需求选择适合的方式进行通信。它们提供了不同的数据交换和通信方式，适用于不同类型的应用场景。根据业务需求和性能要求，选择合适的 RPC 模式可以提高系统的效率和可扩展性。

### 编写.proto 文件
[[散记/gRPC#ProtoBuf|proto]]
.proto 文件可以定义 rpc 调用中的数据交换 `消息`, 也可以定义 rpc `服务`
这里先编写 `createUser, loginUser` 的 RPC api

定义 `User` 消息, 以及 RPC 中参数消息, 以及 RPC 服务
![[Pasted image 20240228203442.png]]
![[Pasted image 20240228203352.png]]
![[Pasted image 20240228203422.png]]
![[Pasted image 20240228203431.png]]

然后使用 protoc 生成对应的代码, 生成的结构, 方法, 接口和服务都是根据 `.proto` 中定义的消息和服务来的

`protoc --proto_path=proto --go_out=pb --go_opt=paths=source_relative --go-grpc_out=pb --go-grpc_opt=paths=source_relative proto/*.proto `

[[散记/gRPC#protoc|gRPC]]

CreateUserRequest, CreateUserResponse
![[Pasted image 20240229124521.png]]
![[Pasted image 20240229124552.png]]

User, 同时还提供了一些方法来访问成员
![[Pasted image 20240229124629.png]]
![[Pasted image 20240229124642.png]]


搭建 gRPC 服务器
[[GoGo/packages/grpc|grpc]]
![[Pasted image 20240229115138.png]]
类似地, gRPC 服务器也有 `config, store, tokenMaker`, 但是需要再新嵌入一个 `pb.UnimplementedSimpleBankServer`, 来保证向前拓展性, 从而实现 `SimpleBankServer` (只是 gRPC 提供的拓展性一致功能)
![[Pasted image 20240229115342.png]]

编写 `CreateUser, LoginUser` 方法, 让搭建的 gRPC 服务器实现 `SimpleBankServer` 接口
[[GoGo/packages/grpc#grpc/status|grpc/status]]
[[GoGo/packages/grpc#grpc/codes|grpc/codes]]
![[Pasted image 20240229124945.png]]
![[Pasted image 20240229124957.png]]
这两个方法的编写与 HTTP JSON 流程都类似, 只是在参数读取, 返回不一样


修改 main.go, 启动 gRPC 服务器
这里使用了 protoc 生成的 `pb.RegisterSimpleBankServer(grpcServer, bankServer)` 进行注册,
相当于 grpc 包接收请求, 并转发到 bankServer, 通过 bankServer 的 handler (上面写的两个)进行处理
![[Pasted image 20240229125136.png]]
[[GoGo/packages/grpc|grpc]]


# 26. 使用 gRPC Gateway 让 gRPC 服务器提供 HTTP JSON 服务
![[Pasted image 20240229152404.png]]
如果是 gRPC 请求, 直接由 gRPC 服务器处理
如果是 HTTP 请求, 通过网关转发到 gRPC 服务器, 然后在通过网关将响应结果转化成 JSON

在服务和 rpc 的 proto 文件中, 给这两个 rpc 新增选项, 
加上 gateway 的路径
![[Pasted image 20240229161734.png]]

编写 gateway server
![[Pasted image 20240229162132.png]]
这段代码是用于运行 gRPC Gateway 服务器的函数，它将 gRPC 服务暴露为 HTTP/JSON 接口。
1. 创建 gRPC 服务器：
   ```go
   server, err := gapi.NewServer(config, store)
   if err != nil {
       log.Fatal("cannot create gRPC server: ", err)
   }
   ```
   首先，根据给定的配置和存储对象，创建一个 gRPC 服务器的实例。

2. 配置 JSON 格式的选项：
   ```go
   jsonOption := runtime.WithMarshalerOption(runtime.MIMEWildcard, &runtime.JSONPb{
       MarshalOptions: protojson.MarshalOptions{
           UseProtoNames: true,
       },
       UnmarshalOptions: protojson.UnmarshalOptions{
           DiscardUnknown: true,
       },
   })
   ```
   使用 `runtime.WithMarshalerOption` 函数，创建一个 JSON 格式的选项对象。它配置了 JSON 序列化和反序列化的选项，包括使用 proto 字段名、忽略未知字段等。

3. 创建 gRPC ServeMux：
   ```go
   gprcMux := runtime.NewServeMux(jsonOption)
   ```
   使用 `runtime.NewServeMux` 函数创建一个 gRPC ServeMux 对象。它是一个 HTTP 请求多路复用器，用于将 HTTP 请求路由到相应的 gRPC 处理程序。
   ==这是 gateway 最重要的一部分, `github.com/grpc-ecosystem/grpc-gateway/v2/runtime.NewServeMux` 返回的是一个实现了 `http.Handler` 接口的多路复用器, 在后续可以正常作为 handler 使用==

4. 注册 gRPC 处理程序：
   ```go
   ctx, cancel := context.WithCancel(context.Background())
   defer cancel()

   err = pb.RegisterSimpleBankHandlerServer(ctx, gprcMux, server)
   if err != nil {
       log.Fatal("cannot register handler server: ", err)
   }
   ```
   使用 `pb.RegisterSimpleBankHandlerServer` 方法将 gRPC 服务器的处理程序注册到 gRPC ServeMux 中。这样，gRPC Gateway 就知道要将 HTTP 请求转发到哪个 gRPC 方法进行处理。 
![[Pasted image 20240229163321.png]]

5. 创建 HTTP ServeMux：
   ```go
   mux := http.NewServeMux()
   mux.Handle("/", gprcMux)
   ```
   使用 `http.NewServeMux` 函数创建一个 HTTP ServeMux 对象。它是一个 HTTP 请求多路复用器，用于将 HTTP 请求路由到相应的处理程序。在这里，我们将所有的请求都路由到之前创建的 gRPC ServeMux（gprcMux）上。

6. 创建监听器并启动 HTTP Gateway 服务器：
   ```go
   listener, err := net.Listen("tcp", config.HTTPServerAddress)
   if err != nil {
       log.Fatal("cannot create listener: ", err)
   }

   log.Printf("start HTTP gateway server at %s", listener.Addr().String())
   err = http.Serve(listener, mux)
   if err != nil {
       log.Fatal("cannot start HTTP gateway server: ", err)
   }
   ```
   使用 `net.Listen` 函数创建一个 TCP 监听器，指定服务器要监听的地址和端口。然后，使用 `http.Serve` 函数启动 HTTP 服务器，将监听器和之前创建的 HTTP ServeMux（mux）绑定在一起，开始接受并处理来自客户端的 HTTP 请求。

通过以上步骤，该函数实现了 gRPC Gateway 服务器的搭建和启动。它将 gRPC 服务转换为 HTTP/JSON 接口，允许非 gRPC 客户端通过 HTTP 请求与 gRPC 服务进行交互。同时，它还提供了 JSON 格式的选项来配置序列化和反序列化的行为。最后，通过创建监听器并启动 HTTP 服务器，该函数让 HTTP Gateway 服务器能够接受和处理来自客户端的 HTTP 请求。

在 main 函数中使用 `go runGatewayServer(config, store)` 启动一个新的协程专门处理 HTTP JSON 请求
是的，根据提供的完整代码来看，程序实际上启动了两个 gRPC 服务器。
在 `main()` 函数中，首先加载配置和连接到数据库，然后创建了一个存储对象 `store`。接下来，通过调用 `go runGatewayServer(config, store)` 启动了一个用于处理 gRPC Gateway 转发的 HTTP/JSON 请求的服务器。同时，继续执行 `runGrpcServer(config, store)` 启动了另一个用于处理 gRPC 客户端请求的 gRPC 服务器。

这样，程序启动了两个服务器：
1. `runGatewayServer()` 函数启动的服务器用于处理 gRPC Gateway 转发的 HTTP/JSON 请求。它将 gRPC 服务暴露为 HTTP 接口，允许非 gRPC 客户端通过 HTTP 请求与 gRPC 服务进行交互。
2. `runGrpcServer()` 函数启动的服务器用于处理 gRPC 客户端的请求。它直接处理来自 gRPC 客户端的 gRPC 请求，并提供相应的服务。
通过同时启动这两个服务器，程序能够同时支持 gRPC 客户端和非 gRPC 客户端（通过 gRPC Gateway 转发）的请求，提供灵活的接口选择。


### 总结

`runtime.NewServeMux` 返回的 `grpcMux` 实际上就类似一个中介, 把 `grpcServer.CreateUser, LoginUser` 通过 `RegisterSimpleBankHandlerServer` 拿来作为自己路由 `/v1/create_user, /v1/login_user` 的处理函数,
由于其本身实现了 `http.Handler` 接口
因此然后再挂载到 http mux 上, 提供 HTTP JSON 服务

# 27. 从 gRPC 元数据中提取有用信息
在 gin 中, `userAgent, clientIP` 已经由 gin 处理好并放进 `context` 中了, 可以直接使用,
但是对于 gRPC, 需要进行特别的处理, 从元数据中获得

[[GoGo/packages/grpc#grpc/peer|grpc/peer]]
[[GoGo/packages/grpc#grpc/metadata|grpc/metadata]]
![[Pasted image 20240229171408.png]]


然后通过这个函数读取使用
![[Pasted image 20240229171750.png]]


# 28. 为 gRPC 服务器使用参数验证
gRPC 服务器目前没有提供参数验证, 因此可能会出现一些问题

[[errdetails]]
通过 `errdetails` 这个包, 可以对请求中的字段进行验证, 并且方便地组织错误的格式
![[Pasted image 20240301103628.png]]


对于创建用户, `username, password, full_name, email` 是必须的, 并且需要符合一定格式
![[Pasted image 20240301103446.png]]

对于用户登录:
![[Pasted image 20240301103824.png]]

![[Pasted image 20240301103405.png]]
这里还用到了 `regexp` 包对用户名进行正则表达式匹配验证
[[regexp]]


# 29. 直接使用 go 代码进行 db migration
可以使用 `migrate` 包提供的 golang api 直接在 golang 代码中进行数据库迁移, 
这样可以有效地在环境变量在 main 函数中被加载到 config 时就进行迁移, 而避免了环境变量被覆盖重写
创建一个 `migration` 对象, 在 `migrationURL` 中指定好迁移文件的位置, 使用 `migration.Up()` 进行迁移
[[migrate]]
![[Pasted image 20240301112829.png]]

从而省去了很多在部署时容器中安装 migrate binary, 执行命令的过程


# 30. 新增更新用户 db api
这段代码是一个 SQL 查询语句，用于更新数据库中的用户信息。下面是对代码的解释：

```sql
-- name: UpdateUser :one
UPDATE users
SET
  hashed_password = COALESCE(sqlc.narg(hashed_password), hashed_password),
  full_name = COALESCE(sqlc.narg(full_name), full_name),
  full_name = COALESCE(sqlc.narg(full_name), full_name)
WHERE
  username = sqlc.arg(username)
RETURNING *;
```

解释如下：

1. `-- name: UpdateUser :one`：这是一个注释，用于标识此 SQL 查询的名称为 "UpdateUser"。": one" 表示此查询期望返回一行结果。

2. `UPDATE users`：这是一个 UPDATE 语句，它将更新名为 "users" 的表中的数据。

3. `SET`：这表示接下来将设置要更新的列的值。

4. `hashed_password = COALESCE(sqlc.narg(hashed_password), hashed_password)`：这是一个赋值语句，将 "hashed_password" 列的值设置为一个表达式的结果。`COALESCE()` 函数用于选择非空值。在这里，它的作用是，如果 "hashed_password" 的新值不为空，则使用新值，否则保持原来的值。

5. `full_name = COALESCE(sqlc.narg(full_name), full_name)`：这是另一个赋值语句，将 "full_name" 列的值设置为一个表达式的结果。同样地，如果新值不为空，则使用新值，否则保持原来的值。

6. `WHERE username = sqlc.arg(username)`：这是一个条件语句，用于指定只在满足条件的行上执行更新操作。在这里，它表示只更新具有指定 "username" 值的行。

7. `RETURNING *`：这表示在更新操作完成后，返回受影响的行作为结果。"*"表示返回所有列的值。

总体而言，这段代码的作用是根据给定的条件更新数据库中的用户信息。它使用了 COALESCE 函数来处理新值为空的情况，并通过 WHERE 子句指定了要更新的行。最后，它返回更新后的行作为结果。

[[COALESCE]]

![[Pasted image 20240301121408.png]]
生成对应的 api 接口, 使用 COALESCE 的好处是, 如果遇到了空值(也就是没有更新这一个字段, 那么它会使用原字段)

![[Pasted image 20240301121523.png]]
并编写对应的测试


# 31. 新增更新用户 gRPC api
为 update user api 编写 `proto` 文件
设置服务的 rpc 以及网关路径
![[Pasted image 20240301144410.png]]
![[Pasted image 20240301144508.png]]

![[Pasted image 20240301144743.png]]
由于在 `proto` 中使用了 `optional` 关键字, 因此生成的可选字段都是 `*string`, 为空代表这个请求中没带这个字段



![[Pasted image 20240301144631.png]]
类似 `CreateUser, LoginUser`, 编写 `UpdateUser` 处理方法
![[Pasted image 20240301144714.png]]




# 32. 为 gRPC 服务器新增 auth
对于 `UpdateUser`, 需要对用户进行验证, 成功后才能修改自己的账号
验证的过程还是使用嵌入到请求的 `auth header` 以及 `access token`
![[Pasted image 20240301154123.png]]
这里 `auth header` 也是通过 `metadata` 传递
[[simplebank-BackendMasterClass#27. 从 gRPC 元数据中提取有用信息]]
提取, 检验, 在 `UpdateUser` 中用上
![[Pasted image 20240301154341.png]]

需要注意的是, 由于不像 gin 那样提供了完备的中间件注册, 不能直接将函数压入到 handler 调用链中
这里需要手动调用

# 33. 给 gRPC 服务器增加 logger

编写 grpc logger:
![[Pasted image 20240301191558.png]]
grpc logger 本质上是一个 `grpc.UnaryServerInterceptor`, 作用就是拦截请求, 然后做一些中间处理, 然后通过参数中的 `handler` 转发, 很适合来做 log 中间件
![[Pasted image 20240301191749.png]]

![[Pasted image 20240301191817.png]]
然后通过这个 grpc logger 函数生成一个 `grpc.ServerOption`, 来创建一个带有 `logger option` 的 `grpc server`

使用 `zerolog` 包提供更易于理解的日志
`Zerolog` 则是为 log 提供了更丰富易于阅读的展示
[[zerolog]]



# 34. 为网关转发的 HTTP 请求也增加 logger

编写将 logger 插入到 handler 链中
![[Pasted image 20240303171439.png]]
通过接收原始 handler, 进行中间 log 输出操作, 将原始 handler 嵌入, 返回一个新的 handler

由于 `handler.ServeHTTP` 没有返回, 难以判断后续原始 handler 执行的状态, 因此编写一个实现了 `http.ResponseWriter` 接口的 `ResponseRecoder`, 增加两个域来记录处理结果
注意 `Write, WriteHeader` 两个方法, 在进行原始操作的同时, 还将结果保存在了结构体的两个域中


# 35. 使用 redis 作为异步处理队列
对于一些客户请求任务, 需要进行异步处理(可能是因为任务比较花时间, 也可能是因为对这个任务有特别的安排)

直接在后台跑 `goroutine` 可能会导致服务器崩溃时未完成的任务失去追踪, 因此使用 redis 作为一个任务分布器队列, 一个个地处理任务是一个更好的选择

现在这里需要进行异步处理的是用户注册是异步发送验证邮件(验证邮件需要调用第三方 api, 可能需要一定的处理时间, 因此先返回给用户, 然后再异步调用发送验证邮件更好)

实现的想法很简单: 
1. 用户注册时, 通过分配器将发送邮件的异步请求加入到 redis 队列中
2. 处理器检测到有请求进入到 redis 队列中, 取出请求
3. 开启 goroutine 处理请求
[[asynq]]
在这样的架构下, 显然处理器成了服务器的一部分, 必须长时运行监听
分配器则嵌入到 server 中, 在创建用户时, 创建异步请求
分配器和处理器都需要连接到 redis, 这个通过 `asynq.RedisClientOpt` 做到


通过分配器将请求加入 redis 队列
![[Pasted image 20240304130923.png]]
通过调整 `asynq.Option`, 可以设置更多处理器处理规则,
这里定义了 `asynq.MaxRetry(), asynq.ProcessIn(), asynq.Queue()`


![[Pasted image 20240304131019.png]]
![[Pasted image 20240304131046.png]]
使用 `asynq.Client.Enqueue...` 就可以通过这个客户端将任务放入 redis 队列中

注册分配器, 运行处理器
由于分配任务的分配器是从用户注册请求后, 
服务器发出的任务请求, 因此需要将 `taskDistributor` 放在 server 中
![[Pasted image 20240304131614.png]]


分配器属于内部服务的一部分, 需要长时运行, 不断检查 redis 任务队列中是否有任务需要处理
![[Pasted image 20240304131647.png]]
![[Pasted image 20240304131750.png]]




# 36. 将数据库创建用户和分配器新增请求作为一个事务进行处理
上面的实现中, 先调用数据库 api 新建用户, 然后再调用 asynq 分配器新增请求, 这样可能会导致用户插入成功了, 但是分配器请求失败

因此使用一个事务将这两个操作包括起来, 一起提交或回滚
这个事务就类似`transferTx`

![[Pasted image 20240304152724.png]]
![[Pasted image 20240304152805.png]]




# 37. 


# 38. 


# 39. 


# 40. 


# 41. 


# 42. 


# 43. 


# 44. 


# 45. 


# 46. 