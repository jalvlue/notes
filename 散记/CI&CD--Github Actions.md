![[Pasted image 20240221231932.png]]

GitHub Actions 提供了一种自动化工作流的方式，用于构建、测试、部署和集成软件项目。通过使用 GitHub Actions，开发人员可以创建自定义的工作流程来响应不同的事件，例如推送代码、拉取请求或发布版本。这些工作流程由一个或多个称为"action"的任务组成，每个任务都是一个独立的脚本或命令。

以下是 GitHub Actions 的一些关键特点：
1. **事件触发**: GitHub Actions 可以根据不同的事件来触发工作流程，例如代码推送、拉取请求、问题创建等。开发人员可以根据需要定义特定事件触发的工作流程。

2. **自定义工作流程**: 使用 GitHub Actions，开发人员可以定义自己的工作流程，以适应特定的软件开发和部署需求。工作流程可以包含多个步骤和任务，并支持并行和串行执行。

3. **丰富的操作库**: GitHub Actions 提供了一个丰富的操作库，其中包含许多预定义的操作，用于构建、测试和部署应用程序。开发人员可以直接使用这些操作或创建自己的自定义操作。

4. **持续集成和部署**: GitHub Actions 可以与持续集成和持续部署（CI/CD）流程集成，使开发人员能够自动化构建、测试和部署他们的应用程序。它可以与各种工具和云平台（如 AWS、Azure 和 Google Cloud）集成。

5. **可视化和日志**: GitHub 提供了一个可视化的工作流程编辑器，使开发人员能够直观地定义和管理工作流程。同时，工作流程的执行日志也会被记录下来，方便排查和分析问题。

通过使用 GitHub Actions，开发人员可以将软件开发过程中的各种任务自动化，从而提高开发效率、确保代码质量，并加速软件交付过程。它是一个强大且灵活的工具，适用于单个开发者、团队和开源项目。

Github Actions 通过一个 `yml` 文件编写配置
示例:
![[Pasted image 20240221232510.png]]

这里就包括了
1. `on`: 设置触发条件
2. `jobs`: 设置任务, 上面只有一个 `test`
3. `steps`: 设置步骤
4. `actions`: 为每一个步骤设置行为, `uses` 表示使用 GitHub 提供的可重用的 actions, `run` 表示使用自己的 actions


完整用例:
```yml
name: ci-test

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

jobs:

  test:
    name: Test
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:12
        env:
          POSTGRES_USER: root
          POSTGRES_PASSWORD: secret
          POSTGRES_DB: simple_bank
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:

    - name: Set up Go 1.x
      uses: actions/setup-go@v2
      with:
        go-version: ^1.15
      id: go

    - name: Check out code into the Go module directory
      uses: actions/checkout@v2

    - name: Install golang-migrate
      run: |
        curl -L https://github.com/golang-migrate/migrate/releases/download/v4.12.2/migrate.linux-amd64.tar.gz | tar xvz
        sudo mv migrate.linux-amd64 /usr/bin/migrate
        which migrate

    - name: Run migrations
      run: make migrateup

    - name: Test
      run: make test
```
这个配置是一个 GitHub Actions 的工作流程，用于构建和测试一个 Golang 项目。下面是对每个部分的解释：

- `name: ci-test`: 这是工作流的名称，可以根据需要进行更改。
- `on`: 定义了触发工作流程的事件。在这个配置中，工作流会在代码推送到 `master` 分支或者创建针对 `master` 分支的 Pull Request 时触发。
- `jobs`: 定义了工作流程中的作业。
- `test`: 这是一个作业的名称，用于测试。
- `runs-on: ubuntu-latest`: 指定作业运行的操作系统环境为最新的 Ubuntu 版本。
- `services`: 定义了在作业运行期间需要启动的服务。在这个配置中，使用了一个名为 `postgres` 的服务，使用 PostgreSQL 12 的 Alpine 镜像，并设置了一些环境变量和端口映射。
- `steps`: 定义了作业中的步骤。
- `Set up Go`: 使用了 `actions/setup-go` 操作来设置 Go 的版本为 1.21。这个操作会下载并配置指定版本的 Go 环境。
- `Check out code into the Go module directory`: 使用了 `actions/checkout` 操作来检出代码到 Go 模块目录。
- `Install golang-migrate`: 使用 `curl` 命令下载并安装 `golang-migrate` 工具。
- `Run migrations`: 执行一个名为 `migrateup` 的 Makefile 任务，用于运行数据库迁移。
- `Test`: 执行一个名为 `test` 的 Makefile 任务，用于运行测试。

总体来说，这个配置会在推送到 `master` 分支或者创建针对 `master` 分支的 Pull Request 时触发一个作业。作业会在 Ubuntu 环境下运行，并启动一个 PostgreSQL 服务。然后，它会设置 Go 环境，检出代码，安装 `golang-migrate` 工具，运行数据库迁移，并执行测试。
