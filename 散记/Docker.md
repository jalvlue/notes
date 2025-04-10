# dockerfile
[[Dockerfile]]

# docker app
![[Pasted image 20240218161832.png]]

![[Pasted image 20240218161934.png]]
`-d` 指的是在后台模式(分离模式)运行容器
`-e` 指的是环境变量, 通常用来设置数据库的用户, 密码


![[Pasted image 20240218162358.png]]
docker 使用独立的端口, 在本机上为了让 docker 可以使用本机进行通信, 需要将本机的一个端口与 docker 容器端口建立映射
使用 `-p`

![[Pasted image 20240218162724.png]]
使用 `exec` 指令可以连接进 docker 容器
`-it` 表示以动态会话方式

由于是有特定目的的容器(例如做数据库的 postgresql), 这些有特定目的容器通常都会提供附带一些常用工具
![[Pasted image 20240219224955.png]]


![[Pasted image 20240218162913.png]]

# docker network
Docker 网络（Docker Network）是 Docker 提供的一种机制，用于创建和管理容器之间的网络通信。通过 Docker 网络，可以将多个容器连接到一个虚拟网络中，使它们能够相互通信，并与外部网络进行交互。

以下是 Docker 网络的一些关键概念和功能：

1. 默认网络：当安装 Docker 时，会自动创建一个名为 `bridge` 的默认网络。这是一个本地的私有网络，允许容器通过网络进行通信。每个容器都可以自动分配一个 IP 地址，并可以使用容器名称进行相互访问。

2. 网络驱动程序：Docker 支持多种网络驱动程序，用于创建不同类型的网络。除了默认的 `bridge` 驱动程序，还有 `host`、`overlay`、`macvlan` 等驱动程序，每个驱动程序都提供了不同的网络功能和配置选项。

3. 网络连接：可以使用 `docker network connect` 命令将容器连接到一个或多个网络。这样，容器就可以在多个网络之间进行通信，并且可以与连接到同一网络的其他容器进行交互。

4. 网络隔离：Docker 网络提供了一定程度的隔离性。默认情况下，容器只能与同一网络中的其他容器进行通信，无法直接访问主机或其他网络。这种隔离性有助于提高安全性和容器之间的隔离程度。

5. 自定义网络：除了默认网络，Docker 还支持创建自定义网络。自定义网络可以根据需求进行配置，如指定子网、网关、IP 范围等。自定义网络使得容器可以在独立的虚拟网络中运行，可以更好地控制容器之间的通信和连接。

6. 外部访问：Docker 网络可以与主机网络进行连接，使得容器可以通过主机的网络接口与外部网络进行通信。通过端口映射或使用网络代理，可以将容器的服务暴露给外部网络，从而实现容器的外部访问。

Docker 网络提供了一种灵活而强大的机制，用于管理容器之间的通信和连接。它使得容器化应用程序可以轻松地创建和扩展，同时保持安全性和隔离性。通过适当配置和使用 Docker 网络，可以构建复杂的容器化应用程序架构，并实现容器之间的高效通信。

### docker network 指令
以下是一些常用的 Docker 网络指令：
1. `docker network create <network_name>`：创建一个自定义网络。可以通过指定不同的参数来配置网络，例如子网、网关等。

2. `docker network ls`：列出所有可用的 Docker 网络。此命令将显示网络的名称、驱动程序类型、作用域等信息。

3. `docker network inspect <network_name>`：查看特定网络的详细信息，包括网络的配置、与其他网络的连接等。

4. `docker network connect <network_name> <container_name>`：将容器连接到指定网络。这使得容器可以与同一网络中的其他容器进行通信。

5. `docker network disconnect <network_name> <container_name>`：从指定网络中断开容器的连接。

6. `docker network rm <network_name>`：删除指定的网络。要删除网络，必须先断开与该网络连接的所有容器。

7. `docker network inspect --format='{{range .Containers}}{{.Name}} {{end}}' <network_name>`：列出连接到指定网络的容器名称。

8. `docker network prune`：清理不再使用的网络。这将删除没有任何容器连接的所有网络。

这些指令可以帮助您创建、管理和操作 Docker 网络。您可以使用它们来创建自定义网络、连接和断开容器、查看网络配置和状态，以及清理不再使用的网络。通过这些指令，您可以有效地管理容器之间的网络通信和连接。


# docker-compose
Docker Compose 是一个用于定义和运行多容器 Docker 应用程序的工具。它使用一个简单的 YAML 文件来配置应用程序的服务、网络和存储等方面，然后可以使用单个命令将整个应用程序的容器一起启动、停止和管理。

Docker Compose 的主要目标是简化容器化应用程序的部署和管理，尤其是由多个容器组成的应用程序。它提供了一种声明式的方式来描述应用程序的结构和依赖关系，使得整个应用程序的环境配置和启动变得更加简单和可重复。

以下是 Docker Compose 的一些关键概念和功能：

1. YAML 配置文件：使用 Docker Compose，可以创建一个名为 `docker-compose.yml` 的 YAML 文件来定义应用程序的服务、网络、存储卷等。该文件描述了每个服务的镜像、环境变量、端口映射、依赖关系等。

2. 服务定义：在 Compose 文件中，每个容器被称为一个服务（Service）。服务可以有不同的角色和功能，例如 Web 服务器、数据库、消息队列等。您可以定义多个服务，并指定它们之间的关系和依赖性。

3. 单一命令操作：通过使用 `docker-compose` 命令，可以一次性启动、停止和管理整个应用程序的所有容器。这样，您可以轻松地管理多个容器之间的依赖关系和交互。

4. 网络和存储卷：Docker Compose 可以自动创建和管理应用程序所需的网络和存储卷。您可以定义自定义网络，并将容器连接到指定的网络中。此外，您还可以指定存储卷的挂载点和配置。

5. 环境变量和参数化配置：使用 Docker Compose，可以轻松地设置和传递环境变量给应用程序的容器。这使得在不同环境中的部署和配置更加灵活。您还可以使用参数化配置来动态地替换 Compose 文件中的值。

Docker Compose 提供了一种简单而强大的方式来定义、管理和运行多容器 Docker 应用程序。它使得容器化应用程序的部署和维护变得更加简单和可靠，同时提供了一致性和可重复性的环境配置。

### docker-compose.yaml 例子
![[Pasted image 20240227162526.png]]
这个 `docker-compose.yaml` 文件定义了两个服务：`postgres` 和 `api`。让我们逐行解释每个部分的含义：

- `version: "3.9"`：指定了使用的 Docker Compose 文件格式的版本。

- `services`：定义了要运行的服务的列表。

  - `postgres`：这是一个服务名称，用于标识要运行的容器。

    - `image: postgres:12-alpine`：使用 `postgres:12-alpine` 镜像作为容器的基础镜像。

    - `environment`：设置了一些环境变量，包括 `POSTGRES_USER`、`POSTGRES_PASSWORD` 和 `POSTGRES_DB`，用于配置 PostgreSQL 数据库的用户名、密码和数据库名。

  - `api`：另一个服务名称。

    - `build`：使用当前目录作为构建上下文，并使用名为 `Dockerfile` 的文件进行构建。

    - `ports`：将主机的 8080 端口映射到容器的 8080 端口，以便可以通过主机的 8080 端口访问 API。

    - `environment`：设置了一个名为 `DB_SOURCE` 的环境变量，用于指定 API 与 PostgreSQL 数据库的连接字符串。

    - `depends_on`：指定了依赖关系，表示该服务依赖于 `postgres` 服务。这意味着在启动 `api` 服务之前，`postgres` 服务必须先启动。

    - `entrypoint`：定义了容器启动时要执行的命令。在这个例子中，它指定了一个脚本 `wait-for.sh` 来等待 `postgres` 服务的可用性，然后再执行 `start.sh` 脚本。

    - `command`：定义了在容器启动时要执行的默认命令。在这里，它指定要运行的主应用程序文件为 `/app/main`。

这个 `docker-compose.yaml` 文件描述了一个由两个服务组成的应用程序：一个是运行 PostgreSQL 数据库的 `postgres` 服务，另一个是构建的 API 服务（使用 Dockerfile 进行构建）。它们之间有依赖关系，API 服务在启动之前需要确保 PostgreSQL 服务已经启动。此外，还定义了一些环境变量和命令，用于配置和运行这两个服务的容器。


### docker-compose 指令
Docker Compose 提供了一组命令（指令）来管理和操作 Compose 文件定义的服务。以下是一些常用的 Docker Compose 指令：

1. `up`: 启动 Compose 文件中定义的服务。如果服务的容器不存在，则会创建并启动它们。如果容器已经存在，则会重新启动它们。可以使用 `-d` 选项在后台运行容器。

   示例：`docker-compose up -d`

2. `down`: 停止 Compose 文件中定义的服务，并删除相关的容器、网络和存储卷等资源。

   示例：`docker-compose down`

3. `start`: 启动 Compose 文件中定义的服务。与 `up` 命令不同，`start` 命令仅启动已经存在的容器，而不会重新创建它们。

   示例：`docker-compose start`

4. `stop`: 停止 Compose 文件中定义的服务。与 `down` 命令不同，`stop` 命令仅停止容器，而不会删除它们。

   示例：`docker-compose stop`

5. `restart`: 重启 Compose 文件中定义的服务。该命令会先停止容器，然后再次启动它们。

   示例：`docker-compose restart`

6. `ps`: 显示 Compose 文件中定义的服务的状态。它会列出每个服务的容器 ID、状态、端口映射等信息。

   示例：`docker-compose ps`

7. `logs`: 查看 Compose 文件中定义的服务的日志输出。可以使用 `-f` 选项实时跟踪日志。

   示例：`docker-compose logs -f`

8. `exec`: 在 Compose 文件中定义的服务的容器中执行命令。

   示例：`docker-compose exec <service_name> <command>`

这些是一些常用的 Docker Compose 指令，用于管理和操作 Compose 文件中定义的服务。可以使用这些指令来启动、停止、重启服务，查看状态和日志，以及在容器中执行命令等。使用 `docker-compose --help` 命令可以查看更多可用的指令和选项。