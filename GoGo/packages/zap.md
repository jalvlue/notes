# Overview
- `zap` 是一个高性能日志库, 


# API
- `zap` 的核心是 `zap.Logger struct`, 提供了所有与日志相关的功能
- 除此之外还有 `zap.Config, zap.Option`, 来自定义 logger 的特性


### New
- `zap` 没有 `global logger` 的概念, 因此使用的时候必须新建
- `zap.NewExample()`
- `zap.Must(zap.NewDevelopment())`
- `zap.Must(zap.NewProduction())`
