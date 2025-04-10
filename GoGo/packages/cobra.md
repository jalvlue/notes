[cobra](https://github.com/spf13/cobra)

- `cobra` 是一个快速构建 cli 应用的库, 核心概念是 `subCommand, flags`

Cobra 是 Go 语言中一个非常流行的库，用于创建命令行应用程序（CLI）。它提供了一个简单而强大的框架，帮助开发者快速构建具有丰富命令和参数的 CLI 工具。以下是 Cobra 的一些关键特点和功能：

### 主要特点

1. **命令结构**：
   - Cobra 允许你轻松定义命令和子命令，形成一个树状结构，使得 CLI 应用的组织更加清晰。

2. **易于使用**：
   - 提供了简单的 API 来创建命令、标志和参数。你只需编写少量代码即可实现复杂的功能。

3. **自动生成帮助信息**：
   - 自动生成命令的帮助和使用信息，用户可以通过 `--help` 选项快速获取指令说明。

4. **支持标志和参数**：
   - 支持全局和命令级别的标志（flags），可以轻松处理命令行参数。

5. **与其他库兼容**：
   - 可以与其他库（如 Viper）结合使用，以处理配置文件和环境变量。

### 示例代码

以下是一个简单的 Cobra 应用示例：

```go
package main

import (
    "fmt"
    "github.com/spf13/cobra"
    "os"
)

func main() {
    var rootCmd = &cobra.Command{
        Use:   "app",
        Short: "A simple CLI application",
        Long:  `A longer description of your application.`,
        Run: func(cmd *cobra.Command, args []string) {
            fmt.Println("Welcome to the CLI app!")
        },
    }

    var greetCmd = &cobra.Command{
        Use:   "greet",
        Short: "Greet someone",
        Run: func(cmd *cobra.Command, args []string) {
            name, _ := cmd.Flags().GetString("name")
            fmt.Printf("Hello, %s!\n", name)
        },
    }

    greetCmd.Flags().StringP("name", "n", "World", "Name to greet")
    rootCmd.AddCommand(greetCmd)
    rootCmd.Execute()
}
```

### 运行示例

1. 编译并运行应用：
   ```bash
   go run main.go
   ```

2. 使用 greet 命令：
   ```bash
   go run main.go greet --name Alice
   ```

### 总结

Cobra 是一个功能强大的库，适合构建复杂的 CLI 应用。它的简单性和灵活性使得开发者能够快速上手。如果你需要创建 CLI 工具，Cobra 是一个极好的选择！

