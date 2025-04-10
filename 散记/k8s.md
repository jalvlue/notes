### Cluster
- `cluster`: k8s 集群, 实际生产环境中, 一个集群可以包括多个实际物理节点
- 节点类型:
	- `Control Plane Node`: 控制平面节点
	- `Worker Node`: 实际运行软件容器化运行时的节点
- `minikube` 可以提供一个本地的单节点集群, 使用 `minikube` 时, 会在 `docker` 中创建一个名为 `minikube` 的容器, 这个容器就是一个单节点 `k8s cluster`, 融合控制平面节点和工作节点为一体
- `k8s cluster` 是一个基础服务
- `minikube start`: 启动/创建集群
- `minikube stop`
- `minikube delete`
- `minikube dashboard --url`: 开启可视化 `dashboard`

### Pod
- `pod`: k8s 的调度基本单位, 其中可以包含一个或多个容器
- 一个或多个 `pod` 运行在一个 `worker node` 中

### Deployment
- `deployment`: k8s 的核心, 是用于管理和部署应用程序的抽象概念对象。
-  **Deployment 的作用**：它定义了应用程序的期望状态，例如 Pod 的数量、更新策略等。
- 一个 `Deployment` 可以在一个 `Kubernetes cluster` 中创建和管理多个 `Pod`。`Deployment` 是操作 `cluster` 基础设施的工具。
- `Deployment` 表达了对应用程序状态的期望, 在现有的集群中管理应用的状态。
- `kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.39 -- /agnhost netexec --http-port=8080`
- `kubectl get deployments`
- `kubectl get pods`
- `kubectl get events`
- `kubectl config view`
- `kubectl logs <pod_name>`

### Service
- `service`: 默认情况下, 一个 `pod` 只能在 `cluster` 内部访问, 为了让外部 ip 也可以访问这个 `pod`, 需要将该 `pod` 暴露成一个服务 `service`
- `kubectl expose deployment hello-node --type=LoadBalancer --port=8080`
- `kubectl get services`
- `minikube service hello-node`: 在浏览器中访问该服务

### Addons
- `addon`: 插件, 内置一些服务提供这些功能
-  `minikube addons list`
- `minikube addons enable metrics-server`
- `minikube addons disable metrics-server`
- `kubectl get pod,svc -n kube-system`

### Clean up
- `kubectl delete service hello-node`
- `kubectl delete deployment hello-node`
- `minikube stop`
- `minikube delete`
