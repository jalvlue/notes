goose 和 sqlc:
go 中用来帮助处理 sql 的包

qq

authentication 的过程:
从 http request header 中拿出 `authentication header`, 得到其中的 aipkey


authentication 只是一个进行操作前的一个查看授权小功能而已, 在这里中只是用来根据 apikey 获取用户信息
