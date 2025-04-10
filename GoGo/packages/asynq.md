asynq 是一个分布式任务队列, 使用 redis 作为内在存储
工作流程:
1. Client 将任务发布到队列中
2. Server 从队列中将任务拿出, 并开启一个 worker goroutine 分别处理每一个任务
3. 多个 worker 并行处理多个任务
asynq 是做异步的有利工具
![[Pasted image 20240304123217.png]]


# 以下是一些使用 Asynq 的例子：

1. 任务队列：您可以使用 Asynq 来构建一个任务队列，用于处理异步任务。例如，您可以将需要后台处理的任务添加到队列中，然后使用 Asynq 的工作进程来并发地处理这些任务。

2. 邮件发送：假设您的应用程序需要发送大量的电子邮件。您可以使用 Asynq 来创建一个邮件发送任务队列。当用户触发发送电子邮件的操作时，您可以将电子邮件相关的数据添加到任务队列中，并使用 Asynq 的工作进程来异步发送电子邮件。

3. 图像处理：如果您有一个需要处理大量图像的应用程序，您可以使用 Asynq 来处理图像处理任务。当用户上传图像时，您可以将图像处理任务添加到队列中，然后使用 Asynq 的工作进程来异步处理这些任务，例如生成缩略图、应用滤镜等。

4. 后台数据处理：对于大规模的数据处理任务，使用 Asynq 可以提高应用程序的响应性能。您可以将数据处理任务添加到队列中，然后使用 Asynq 的工作进程来并发地处理这些任务，从而加快处理速度并减轻主应用程序的负载。

5. 定时任务：Asynq 支持任务调度功能，您可以使用它来创建定时任务。例如，您可以创建一个定时任务来执行备份操作、生成报告等。

这些只是 Asynq 的一些使用例子，实际上，您可以根据应用程序的需求和业务逻辑，将 Asynq 应用于各种不同的场景和任务处理中。


# 以下是一个使用 Asynq API 接口的简单示例：

```go
package main

import (
	"fmt"
	"log"
	"time"

	"github.com/go-redis/redis/v8"
	"github.com/hibiken/asynq"
)

func main() {
	// 创建一个Redis连接
	redisClient := redis.NewClient(&redis.Options{
		Addr:     "localhost:6379",
		Password: "", // 如果有密码，请提供密码
		DB:       0,  // 选择要使用的数据库
	})

	// 创建一个Asynq客户端
	client := asynq.NewClient(asynq.RedisClientOpt(redisClient))

	// 定义一个任务处理函数
	processTask := func(ctx context.Context, task *asynq.Task) error {
		// 在这里编写任务的处理逻辑
		fmt.Println("处理任务:", task.Type, "数据:", string(task.Payload))

		// 模拟任务处理耗时
		time.Sleep(time.Second * 2)

		return nil
	}

	// 注册任务处理函数
	handler := asynq.HandlerFunc(processTask)
	client.SetDefaultHandler(handler)

	// 创建一个任务
	task := asynq.NewTask("send_email", []byte(`{"to": "example@example.com", "message": "Hello, World!"}`))

	// 将任务添加到队列中
	info, err := client.Enqueue(task)
	if err != nil {
		log.Fatal("无法添加任务:", err)
	}

	fmt.Println("任务已添加到队列，任务ID:", info.ID)
}
```

在上述示例中，我们首先创建了一个 Redis 连接，然后使用 `asynq.NewClient` 函数创建了一个 Asynq 客户端。接下来，我们定义了一个任务处理函数 `processTask`，该函数接收一个任务并处理它。在这个示例中，我们简单地打印任务类型和有效负载，并模拟任务处理耗时。

然后，我们使用 `asynq.HandlerFunc` 将任务处理函数注册到 Asynq 客户端。接下来，我们创建了一个任务，并使用 `client.Enqueue` 方法将任务添加到队列中。最后，我们打印了任务的 ID。

请注意，这只是一个简单的示例，用于说明如何使用 Asynq 的 API 接口。您可以根据实际需求和业务逻辑编写更复杂的任务处理代码。此外，还可以使用 Asynq 提供的其他功能，如任务调度、任务重试等，以满足您的需求。
