当使用 `github.com/stretchr/testify/require` 库时，以下是一些示例用法：

1. 基本断言：
   ```go
   require.Equal(t, 42, CalculateAnswer())
   require.True(t, IsLoggedIn())
   require.Contains(t, []string{"apple", "banana", "orange"}, "banana")
   ```

2. 嵌套断言：
   ```go
   result := CalculateResult()
   require.Equal(t, expectedValue, result.Value)
   require.Equal(t, expectedSubValue, result.SubValue)
   ```

3. 文件和目录断言：
   ```go
   require.FileExists(t, "/path/to/file")
   require.DirExists(t, "/path/to/directory")
   require.FileMode(t, "/path/to/file", os.ModePerm)
   ```

4. 错误断言：
   ```go
   err := SomeFunction()
   require.Error(t, err)
   require.EqualError(t, err, "expected error message")
   require.ContainsError(t, err, "partial error message")
   ```

5. 跳过和标记测试：
   ```go
   if !condition {
       t.Skip("Skipping test due to condition not met")
   }
   if testing.Short() {
       t.Skip("Skipping test in short mode")
   }
   t.Run("SomeSubTest", func(t *testing.T) {
       t.SkipNow()
   })
   ```

这些示例演示了 `github.com/stretchr/testify/require` 库的一些常见用法。您可以根据您的具体测试需求和断言场景使用这些功能和函数。请注意，这只是一小部分示例，`require` 库还提供了许多其他有用的函数和功能，可以根据您的测试需求进行探索和使用。
