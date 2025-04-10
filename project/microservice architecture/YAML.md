`Kubernetes` 使用 `YAML` 作为其主要配置语言

在 `k8s` 中, `yaml` 通常被用于几种对象, 在 `yaml` 文件的 `kind` 属性中定义
主要的 `k8s` 对象有: `Pod`, `Deployment`, `Service`, `ConfigMap`, `Secret` 等等等, 不同对象分别有什么作用???

### 简单例子
`auth-deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth
  labels:
    app: auth
spec:
  replicas: 2
  selector:
    matchLabels:
      app: auth
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 3
  template:
    metadata:
      labels:
        app: auth
    spec:
      containers:
        - name: auth
          image: jalvlue/auth
          ports:
            - containerPort: 5000
          envFrom:
            - configMapRef:
                name: auth-configmap
            - secretRef:
                name: auth-secret

```

这是一个简单的 Kubernetes 部署配置文件，用于定义一个名为 "auth" 的部署。
- `apiVersion: apps/v1`：指定使用的 Kubernetes API 版本为 apps/v 1，表示这是一个应用程序相关的配置。
- `kind: Deployment`：定义了这个配置文件描述的 Kubernetes 对象类型为 Deployment，即部署对象。
- `metadata`：包含了与 Deployment 相关的元数据信息，如名称和标签。
  - `name: auth`：指定部署的名称为 "auth"。
  - `labels`：为 Deployment 添加了一个标签，标签的键为 "app"，值为 "auth"。这个标签可以用于识别和选择相关的资源。
- `spec`：定义了 Deployment 的规格和配置。
  - `replicas: 2`：指定需要创建的 Pod 副本数为 2，即希望在集群中运行 2 个相同的 Pod 实例。
  - `selector`：指定了用于选择要管理的 Pod 的标签选择器。
    - `matchLabels`：定义了一个标签选择器，选择具有 "app: auth" 标签的 Pod。
  - `strategy`：定义了部署的更新策略。
    - `type: RollingUpdate`：指定了使用滚动更新策略进行部署，即逐步更新 Pod 实例，确保服务不中断。
    - `rollingUpdate`：指定了滚动更新的相关配置。
      - `maxSurge: 3`：指定了在滚动更新期间允许的最大超出副本数为 3，即最多可以同时创建 3 个新的 Pod 实例。
  - `template`：定义了要创建的 Pod 的模板。
    - `metadata`：指定了 Pod 模板的元数据信息。
      - `labels`：为 Pod 模板添加了一个标签，标签的键为 "app"，值为 "auth"。这个标签可以用于识别和选择相关的资源。
    - `spec`：定义了 Pod 的规格和配置。
      - `containers`：定义了在 Pod 中运行的容器的相关配置。
        - `name: auth`：指定容器的名称为 "auth"。
        - `image: jalvlue/auth`：指定容器要使用的镜像。
        - `ports`：定义了容器需要暴露的端口。
          - `containerPort: 5000`：指定容器监听的端口号为 5000。
        - `envFrom`：指定容器的环境变量来源。
          - `configMapRef`：引用了一个名为 "auth-configmap" 的 ConfigMap，将该 ConfigMap 中的环境变量注入到容器中。
          - `secretRef`：引用了一个名为 "auth-secret" 的 Secret，将该 Secret 中的环境变量注入到容器中。

这个配置文件描述了一个 `Deployment` 对象，它将创建两个名为 "auth" 的 `Pod` 实例，每个 `Pod` 实例都运行一个名为 "auth" 的容器，使用镜像 "jalvlue/auth"。容器监听端口号 5000，并从一个 `ConfigMap` 和一个 `Secret` 中获取环境变量。这个 `Deployment` 对象还定义了滚动更新策略，确保在更新期间最多同时创建 3 个新的 `Pod` 实例。


`configmap.yaml`
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: auth-configmap
data:
  MYSQL_HOST: host.minikube.internal
  MYSQL_USER: auth_user
  MYSQL_DB: auth
  MYSQL_PORT: "3306"
```
这是一个名为 "configmap. Yaml" 的 YAML 文件，用于创建一个 ConfigMap 对象，用于存储认证服务的配置信息。下面是对该文件的解释：
- `apiVersion: v1`：指定使用的 Kubernetes API 版本为 v 1，这是 ConfigMap 对象定义的 API 版本。
- `kind: ConfigMap`：指定要创建的对象类型是 ConfigMap，表示这个 YAML 文件描述的是一个 ConfigMap 对象。
- `metadata`：ConfigMap 对象的元数据部分，包含一些关于对象的信息。
  - `name: auth-configmap`：指定 ConfigMap 的名称为 "auth-configmap"，这是 ConfigMap 在 Kubernetes 中的唯一标识。
- `data`：ConfigMap 对象的数据部分，用于存储具体的配置信息。
  - `MYSQL_HOST: host.minikube.internal`：定义了一个名为 "MYSQL_HOST" 的键值对，键是 "MYSQL_HOST"，值是 "host. Minikube. Internal"。这表示认证服务使用的 MySQL 主机地址。
  - `MYSQL_USER: auth_user`：定义了一个名为 "MYSQL_USER" 的键值对，键是 "MYSQL_USER"，值是 "auth_user"。这表示认证服务使用的 MySQL 用户名。
  - `MYSQL_DB: auth`：定义了一个名为 "MYSQL_DB" 的键值对，键是 "MYSQL_DB"，值是 "auth"。这表示认证服务使用的 MySQL 数据库名称。
  - `MYSQL_PORT: "3306"`：定义了一个名为 "MYSQL_PORT" 的键值对，键是 "MYSQL_PORT"，值是 "3306"。注意，这里使用了双引号将值包裹起来，表示值是一个字符串。

这个 ConfigMap 可以在 Kubernetes 中使用，其他的 Pod 或容器可以通过引用 ConfigMap 来获取其中的配置信息，例如在认证服务的 Pod 中使用环境变量或配置文件来获取 MySQL 的主机地址、用户名、数据库名称和端口号等信息。


`secret.yaml`
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: auth-secret
stringData:
  MYSQL_ROOT_PASSWORD: Auth123
  JWT_SECRET: sarcasm
type: Opaque
```
这是一个名为 "secret. Yaml" 的 YAML 文件，用于创建一个 Secret 对象，用于存储敏感的认证相关信息。下面是对该文件的解释：
- `apiVersion: v1`：指定使用的 Kubernetes API 版本为 v 1，这是 Secret 对象定义的 API 版本。
- `kind: Secret`：指定要创建的对象类型是 Secret，表示这个 YAML 文件描述的是一个 Secret 对象。
- `metadata`：Secret 对象的元数据部分，包含一些关于对象的信息。
  - `name: auth-secret`：指定 Secret 的名称为 "auth-secret"，这是 Secret 在 Kubernetes 中的唯一标识。
- `stringData`：Secret 对象的数据部分，用于存储具体的敏感信息。
  - `MYSQL_ROOT_PASSWORD: Auth123`：定义了一个名为 "MYSQL_ROOT_PASSWORD" 的键值对，键是 "MYSQL_ROOT_PASSWORD"，值是 "Auth 123"。这表示认证服务使用的 MySQL 的 root 密码。
  - `JWT_SECRET: sarcasm`：定义了一个名为 "JWT_SECRET" 的键值对，键是 "JWT_SECRET"，值是 "sarcasm"。这表示认证服务使用的 JSON Web Token (JWT) 的密钥。
- `type: Opaque`：指定 Secret 的类型为 "Opaque"，表示这是一个通用类型的 Secret，可以存储任意类型的数据。
这个 Secret 可以在 Kubernetes 中使用，其他的 Pod 或容器可以通过引用 Secret 来获取其中的敏感信息，例如在认证服务的 Pod 中使用环境变量或配置文件来获取 MySQL 的 root 密码和 JWT 密钥等信息。Secret 对象会以加密的方式存储在 Kubernetes 中，确保敏感信息的安全性。


`service.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: auth
spec:
  selector:
    app: auth
  type: ClusterIP
  ports:
    - port: 5000
      targetPort: 5000
      protocol: TCP
```
这是一个名为 "service. Yaml" 的 YAML 文件，用于创建一个 Service 对象，用于定义 Kubernetes 集群中的服务。下面是对该文件的解释：
- `apiVersion: v1`：指定使用的 Kubernetes API 版本为 v 1，这是 Service 对象定义的 API 版本。
- `kind: Service`：指定要创建的对象类型是 Service，表示这个 YAML 文件描述的是一个 Service 对象。
- `metadata`：Service 对象的元数据部分，包含一些关于对象的信息。
  - `name: auth`：指定 Service 的名称为 "auth"，这是 Service 在 Kubernetes 中的唯一标识。
- `spec`：Service 对象的规格部分，定义了服务的规格和行为。
  - `selector`：指定该 Service 所选择的 Pod 的标签。
    - `app: auth`：使用标签选择器指定选择具有 "app=auth" 标签的 Pod。这意味着该 Service 将路由流量到具有该标签的 Pod。
  - `type: ClusterIP`：定义了 Service 的类型为 "ClusterIP"，这表示 Service 将获得一个集群内部的 IP 地址，并且只能从集群内部访问。
  - `ports`：定义了 Service 的端口配置。
    - `port: 5000`：指定了 Service 的端口号为 5000，这是 Service 对外公开的端口号。
    - `targetPort: 5000`：指定了 Service 将流量转发到 Pod 的端口号为 5000。这意味着 Service 将将外部流量转发到具有 "app=auth" 标签的 Pod 的端口号为 5000 的容器。
    - `protocol: TCP`：指定了流量的协议为 TCP。
这个 Service 可以在 Kubernetes 中使用，它将为具有 "app=auth" 标签的 Pod 提供一个 ClusterIP 地址，并公开端口号为 5000 的服务。其他的 Pod 或容器可以通过该 Service 的 ClusterIP 地址和端口号来访问具有 "app=auth" 标签的 Pod 提供的服务。



这是对四个 YAML 文件的总结：
1. `configmap.yaml`：创建了一个 ConfigMap 对象，用于存储认证服务的配置信息，例如 MySQL 主机地址、用户名、数据库名称和端口号等。ConfigMap 可以被其他 Pod 或容器引用，用于获取配置信息。
2. `secret.yaml`：创建了一个 Secret 对象，用于存储敏感的认证相关信息，例如 MySQL 的 root 密码和 JWT 密钥等。Secret 可以被其他 Pod 或容器引用，用于获取敏感信息，并确保数据的安全性。
3. `service.yaml`：创建了一个 Service 对象，定义了 Kubernetes 集群中的服务。该 Service 使用 ClusterIP 类型，将具有 "app=auth" 标签的 Pod 通过端口号 5000 访问，并且只能从集群内部访问。
4. `auth-deployment.yaml`：创建了一个 Deployment 对象，用于在 Kubernetes 中运行认证服务。该 Deployment 定义了两个副本的 Pod，使用滚动更新策略，并指定了容器镜像、端口配置以及从 ConfigMap 和 Secret 中获取的环境变量。
通过这四个 YAML 文件，你可以在 Kubernetes 中配置和管理认证服务。ConfigMap 和 Secret 用于存储配置和敏感信息，Service 对象定义了服务的访问方式，而 Deployment 对象管理认证服务的运行和扩展。这些文件提供了一种声明性的方式来定义和部署应用程序和服务，使得管理和维护 Kubernetes 集群变得更加方便和可靠。



