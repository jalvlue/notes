# best practice 1: define interfaces where they are used
- Loose coupling
- Consumer package
```go
type FileReader interface {
	ReadFile(filePath string) ([]byte, error)
}

func ProcessFile(reader FileReader, filePath string) ([]byte, error) {
	return reader.ReadFile(filePath)
}
```
- Implementation package
```go
type LocalFileReader struct {}

func (lfr LocalFileReader) ReadFile(filePath string) ([]byte, error) {
	// implementation
}
```
- Dependency inversion![[Pasted image 20241023095446.png]]


# best practice 2: keep interfaces small and focused
- Interface segregation principle
- 最小功能, 简单, 易懂, 如果有需要的话可以使用接口组合来将多个接口的功能合并
```go
type FileSaver interface {
	SaveFile(filePath string, data []byte) error
}

type FileRetriever interface {
	RetrieveFile(filePath string) ([]byte, error)
}
```


# best practice 3: use composition to build more complex interfaces
- 在最小功能接口的前提之下, 如果需要更加复杂的接口则可以使用接口组合
```go
type FileSaver interface {
	SaveFile(filePath string, data []byte) error
}

type FileRetriever interface {
	RetrieveFile(filePath string) ([]byte, error)
}

type FileManager interface {
	FileSaver
	FileRetriever
}
```
-  ![[Pasted image 20241023100235.png]]

# best practice 4: understand the zero value of interfaces
- 空接口和接口引用的实现为空是不一样的
```go
var fileManager FileManager // Uninitialized interface, nil

type CloudFileManager struct{}

func (cfm CloudFileManager) SaveFile(filePath string, data []byte) error {
    // Method implementation
}

func (cfm CloudFileManager) RetrieveFile(filePath string) ([]byte, error) {
    // Method implementation
}

var cfm *CloudFileManager
fileManager = cfm

if fileManager != nil {
    fileManager.SaveFile("path/to/file", []byte("data"))
}
```


# best practice 5: design for interface, not implementation
- 要想着接口应该能提供怎样的行为和功能, 不要想实现类的事
- 