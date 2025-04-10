![[Pasted image 20231003225111.png]]

```shell
├── api
├── assets
├── build
│   ├── ci 
│   └── package 
├── cmd 
│   └── _your_app_ 
├── configs 
├── deployments 
├── docs 
├── examples 
├── githooks 
├── init 
├── internal 
│   ├── app 
│   │   └── _your_app_ 
│   └── pkg 
│       └── _your_private_lib_ 
├── pkg 
│   └── _your_public_lib_ 
├── scripts 
├── test 
├── third_party 
├── tools
├── vendor
├── web
│   ├── app
│   ├── static
│   └── template
├── website
├── .gitignore
├── LICENSE.md
├── Makefile
├── README.md
└── go.mod
```


在一个 go 项目中, /cmd, /pkg, /internal 是最重要的三个子目录, 其他目录可以按需创建

/cmd 一般是项目的入口, 包含 main.go
/internal 存放私有应用和库代码
/pkg 存放外部依赖, 可直接用 import 导入的代码
