# Overview
- 软件设计模式是反复使用, 多人知晓, 的代码设计经验总结
- 使用设计模式是为了代码可重用, 让代码更容易被他人理解并保证代码可重用性
- 用固定套路解决编程问题
- 有哪些设计模式
	- 创建型: 如何创建对象
	- 结构性: 如何实现类或对象的组合
	- 行为型: 类或对象如何交互以及如何分配职责


# 设计原则
- 高内聚, 低耦合

### 单一职责原则


### 开闭原则


### 里氏代换原则


### 依赖倒置原则
![[Pasted image 20241027105850.png]]


### 接口隔离原则
- 使用简单的接口
- 在有必要的时候再将简单接口通过组合获得复杂的接口


### 合成复用原则
- 组合优于继承
- `go` 中只有组合, 没有这种烦恼


### 迪米特法则
- 一个对象应该对其他对象了解得足够少, 从而降低耦合度
- 通常通过一个小接口来将对象之间必要的调用抽象出来


# 创建型模式
- 创建对象的模式

### 单例模式
- 懒汉: 启动时不实例化单例, 等到第一次使用时才去实例化全局单例
- 饿汉: 启动时就去实例化
```go
package singleton

// Singleton 饿汉式单例
type Singleton struct {
	a string // 测试字段
}

var singleton *Singleton

func init() {
	singleton = &Singleton{a: "test"}
}

// GetInstance 获取实例
func GetInstance() *Singleton {
	return singleton
}

var (
	lazySingleton *Singleton
	once          = &sync.Once{}
)

// GetLazyInstance 懒汉式
func GetLazyInstance() *Singleton {
	// once 内的方法只会执行一次，所以不需要再次判断
	once.Do(func() {
		lazySingleton = &Singleton{a: "test"}
	})

	return lazySingleton
}

```

### 简单工厂
- 实际上就是将根据注入的信息, 利用接口抽象的表达能力来创建多种实际类型
```go
package factory

// IRuleConfigParser IRuleConfigParser
type IRuleConfigParser interface {
	Parse(data []byte)
}

// jsonRuleConfigParser jsonRuleConfigParser
type jsonRuleConfigParser struct {
}

// Parse Parse
func (J jsonRuleConfigParser) Parse(data []byte) {
	panic("implement me")
}

// yamlRuleConfigParser yamlRuleConfigParser
type yamlRuleConfigParser struct {
}

// Parse Parse
func (Y yamlRuleConfigParser) Parse(data []byte) {
	panic("implement me")
}

// NewIRuleConfigParser NewIRuleConfigParser
func NewIRuleConfigParser(t string) IRuleConfigParser {
	switch t {
	case "json":
		return jsonRuleConfigParser{}
	case "yaml":
		return yamlRuleConfigParser{}
	}
	return nil
}

```

### 工厂方法
- 实际上就是在简单工厂的基础之上, 将工厂提炼出来, 让每一种工厂去构建一种实际类型, 总的工厂不直接返回产品, 而是返回一个具体工厂, 让调用方通过该具体工厂来获取产品
```go
package factory

// IRuleConfigParser IRuleConfigParser
type IRuleConfigParser interface {
	Parse(data []byte)
}

// jsonRuleConfigParser jsonRuleConfigParser
type jsonRuleConfigParser struct {
}

// Parse Parse
func (J jsonRuleConfigParser) Parse(data []byte) {
	panic("implement me")
}

// yamlRuleConfigParser yamlRuleConfigParser
type yamlRuleConfigParser struct {
}

// Parse Parse
func (Y yamlRuleConfigParser) Parse(data []byte) {
	panic("implement me")
}

// IRuleConfigParserFactory 工厂方法接口
type IRuleConfigParserFactory interface {
	CreateParser() IRuleConfigParser
}

// yamlRuleConfigParserFactory yamlRuleConfigParser 的工厂类
type yamlRuleConfigParserFactory struct {
}

// CreateParser CreateParser
func (y yamlRuleConfigParserFactory) CreateParser() IRuleConfigParser {
	return yamlRuleConfigParser{}
}

// jsonRuleConfigParserFactory jsonRuleConfigParser 的工厂类
type jsonRuleConfigParserFactory struct {
}

// CreateParser CreateParser
func (j jsonRuleConfigParserFactory) CreateParser() IRuleConfigParser {
	return jsonRuleConfigParser{}
}

// NewIRuleConfigParserFactory 用一个简单工厂封装工厂方法
func NewIRuleConfigParserFactory(t string) IRuleConfigParserFactory {
	switch t {
	case "json":
		return jsonRuleConfigParserFactory{}
	case "yaml":
		return yamlRuleConfigParserFactory{}
	}
	return nil
}

```


### 抽象工厂
- 实际上跟工厂方法非常类似, 只是具体工厂可以通过多种不同的方法构建多种具体实现类(在抽象工厂级别的具体实现类一般都是非常接近的了)
```go
package factory

// IRuleConfigParser IRuleConfigParser
type IRuleConfigParser interface {
	Parse(data []byte)
}

// jsonRuleConfigParser jsonRuleConfigParser
type jsonRuleConfigParser struct{}

// Parse Parse
func (j jsonRuleConfigParser) Parse(data []byte) {
	panic("implement me")
}

// ISystemConfigParser ISystemConfigParser
type ISystemConfigParser interface {
	ParseSystem(data []byte)
}

// jsonSystemConfigParser jsonSystemConfigParser
type jsonSystemConfigParser struct{}

// Parse Parse
func (j jsonSystemConfigParser) ParseSystem(data []byte) {
	panic("implement me")
}

// IConfigParserFactory 工厂方法接口
type IConfigParserFactory interface {
	CreateRuleParser() IRuleConfigParser
	CreateSystemParser() ISystemConfigParser
}

type jsonConfigParserFactory struct{}

func (j jsonConfigParserFactory) CreateRuleParser() IRuleConfigParser {
	return jsonRuleConfigParser{}
}

func (j jsonConfigParserFactory) CreateSystemParser() ISystemConfigParser {
	return jsonSystemConfigParser{}
}

type yamlConfigParserFactory struct{}

func (y yamlConfigParserFactory) CreateRuleParser() IRuleConfigParser {
	return yamlRuleConfigParser{}
}

func (y yamlConfigParserFactory) CreateSystemParser() ISystemConfigParser {
	return yamlSystemConfigParser{}
}

type yamlRuleConfigParser struct{}

func (y yamlRuleConfigParser) Parse(data []byte) {
	panic("implement me")
}

type yamlSystemConfigParser struct{}

func (y yamlSystemConfigParser) ParseSystem(data []byte) {
	panic("implement me")
}

// NewIConfigParserFactory ...
// 抽象工厂基于工厂方法，用一个简单工厂封装工厂方法
// 只不过一个工厂方法可创建多个相关的类
func NewIConfigParserFactory(t string) IConfigParserFactory {
	switch t {
	case "json":
		return jsonConfigParserFactory{}
	case "yaml":
		return yamlConfigParserFactory{}
	}
	return nil
}

```


### 建造者模式
- 参数很多的时候, 可以让使用方通过注入设置函数来按需设置
```go
package builder

import "fmt"

type ResourcePoolConfigFunc func(o *ResourcePoolConfigBuilder) error

func WithName(name string) ResourcePoolConfigFunc {
	return func(o *ResourcePoolConfigBuilder) error {
		return o.SetName(name)
	}
}

func WithMinIdle(minIdle int) ResourcePoolConfigFunc {
	return func(o *ResourcePoolConfigBuilder) error {
		return o.SetMinIdle(minIdle)
	}
}

func WithMaxIdle(maxIdle int) ResourcePoolConfigFunc {
	return func(o *ResourcePoolConfigBuilder) error {
		return o.SetMaxTotal(maxIdle)
	}
}

func WithMaxTotal(maxTotal int) ResourcePoolConfigFunc {
	return func(o *ResourcePoolConfigBuilder) error {
		return o.SetMaxTotal(maxTotal)
	}
}

func (b *ResourcePoolConfigBuilder) BuildMy(opts ...ResourcePoolConfigFunc) (*ResourcePoolConfig, error) {
	for _, opt := range opts {
		err := opt(b)
		if err != nil {
			return nil, err
		}
	}
	if b.name == "" {
		return nil, fmt.Errorf("name can not be empty")
	}

	// 设置默认值
	if b.minIdle == 0 {
		b.minIdle = defaultMinIdle
	}

	if b.maxIdle == 0 {
		b.maxIdle = defaultMaxIdle
	}

	if b.maxTotal == 0 {
		b.maxTotal = defaultMaxTotal
	}

	if b.maxTotal < b.maxIdle {
		return nil, fmt.Errorf("max total(%d) cannot < max idle(%d)", b.maxTotal, b.maxIdle)
	}

	if b.minIdle > b.maxIdle {
		return nil, fmt.Errorf("max idle(%d) cannot < min idle(%d)", b.maxIdle, b.minIdle)
	}

	return &ResourcePoolConfig{
		name:     b.name,
		maxTotal: b.maxTotal,
		maxIdle:  b.maxIdle,
		minIdle:  b.minIdle,
	}, nil
}

```


### 原型模式
```go
package prototype

import (
	"encoding/json"
	"time"
)

// Keyword 搜索关键字
type Keyword struct {
	word      string
	visit     int
	UpdatedAt *time.Time
}

// Clone 这里使用序列化与反序列化的方式深拷贝
func (k *Keyword) Clone() *Keyword {
	var newKeyword Keyword
	b, _ := json.Marshal(k)
	json.Unmarshal(b, &newKeyword)
	return &newKeyword
}

// Keywords 关键字 map
type Keywords map[string]*Keyword

// Clone 复制一个新的 keywords
// updatedWords: 需要更新的关键词列表，由于从数据库中获取数据常常是数组的方式
func (words Keywords) Clone(updatedWords []*Keyword) Keywords {
	newKeywords := Keywords{}

	for k, v := range words {
		// 这里是浅拷贝，直接拷贝了地址
		newKeywords[k] = v
	}

	// 替换掉需要更新的字段，这里用的是深拷贝
	for _, word := range updatedWords {
		newKeywords[word.word] = word.Clone()
	}

	return newKeywords
}

```


# 结构型

### 代理模式
- 代理一般是在原有类的基础上需要增加一些需求, 通过代理类来增加, 不改变原有的实现类
- 代理可以分为动态代理和静态代理, go 中一般使用静态代理
![[Pasted image 20241027161500.png]]

```go
package proxy

import (
	"log"
	"time"
)

// IUser IUser
type IUser interface {
	Login(username, password string) error
}

// User 用户
type User struct {
}

// Login 用户登录
func (u *User) Login(username, password string) error {
	// 不实现细节
	return nil
}

// UserProxy 代理类
type UserProxy struct {
	user *User
}

// NewUserProxy NewUserProxy
func NewUserProxy(user *User) *UserProxy {
	return &UserProxy{
		user: user,
	}
}

// Login 登录，和 user 实现相同的接口
func (p *UserProxy) Login(username, password string) error {
	// before 这里可能会有一些统计的逻辑
	start := time.Now()

	// 这里是原有的业务逻辑
	if err := p.user.Login(username, password); err != nil {
		return err
	}

	// after 这里可能也有一些监控统计的逻辑
	log.Printf("user login cost time: %s", time.Now().Sub(start))

	return nil
}
```


### 桥接模式
![[Pasted image 20241027161746.png]]
