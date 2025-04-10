# 例子
`Dockerfile` 定义了构建容器的规则
```Dockerfile
FROM python:3.10-slim-bullseye

RUN echo "">/etc/sources.list \
	&& echo "deb [http://mirrors.ustc.edu.cn/debian/](http://mirrors.ustc.edu.cn/debian/) buster main">>/etc/sources.list \
	&& echo "deb [http://mirrors.ustc.edu.cn/debian/debian-security](http://mirrors.ustc.edu.cn/debian/debian-security) buster/updates main">>/etc/sources.list \
	&& echo "deb [http://mirrors.ustc.edu.cn/debian/debian](http://mirrors.ustc.edu.cn/debian/debian) buster-updates main">>/etc/sources.list \
	&& apt-get update \
	&& apt-get install -y --no-install-recommends --no-install-suggests \
	build-essential default-libmysqlclient-dev \
	&& pip install --no-cache-dir --upgrade pip

WORKDIR /app  
COPY ./requirements.txt /app  
RUN pip install --no-cache-dir --requirement /app/requirements.txt  
COPY . /app

EXPOSE 5000

CMD [ "python3", "server.py" ]
```

这个 Dockerfile 是用来构建一个基于 Python 3.10 的容器镜像，其中安装了一些必要的软件和依赖项，并设置了容器的工作目录和启动命令。
1. `FROM python:3.10-slim-bullseye`：这一行指定了基础镜像，即使用了 Python 3.10 版本的 `slim-bullseye` 镜像，其中包含了基本的 Python 运行环境。
2. `RUN echo "">/etc/sources.list \ ...`：这一系列命令用于修改容器的软件源为中国科技大学的镜像源，以加快软件包的下载速度。其中，`/etc/sources.list` 文件被清空并添加了三个镜像源地址。
3. `apt-get update`：这个命令用于更新容器中的软件包列表，以获取最新的软件包信息。
4. `apt-get install -y --no-install-recommends --no-install-suggests \ build-essential default-libmysqlclient-dev`：这个命令使用 `apt-get` 安装了一些必要的软件包，包括 `build-essential`（构建软件所需的基本工具），`default-libmysqlclient-dev`（MySQL 客户端开发库）等。
5. `pip install --no-cache-dir --upgrade pip`：这个命令使用 `pip` 安装了最新版本的 `pip` 工具。
6. `WORKDIR /app`：这个命令设置容器的工作目录为 `/app`，即后续的命令和操作都将在该目录下进行。
7. `COPY ./requirements.txt /app`：这个命令将当前目录下的 `requirements.txt` 文件复制到容器的 `/app` 目录下。
8. `RUN pip install --no-cache-dir --requirement /app/requirements.txt`：这个命令使用 `pip` 安装了 `requirements.txt` 文件中列出的依赖项，以满足应用程序的运行需求。
9. `COPY . /app`：这个命令将当前目录下的所有文件和文件夹复制到容器的 `/app` 目录下。
10. `EXPOSE 5000`：这个命令声明容器将监听的端口号为 5000，允许外部访问该端口。
11. `CMD [ "python3", "server.py" ]`：这个命令设置容器启动时的默认命令，即在容器内部执行 `python3 server.py`，启动名为 `server.py` 的 Python 应用程序。

在这个目录下执行 `docker build .` 就可以根据这个 `Dockerfile` 生成一个容器了

# Dockerfile 其他关键字
`Dockerfile` 是用于定义和构建 `Docker` 镜像的文本文件，其中包含了一系列的指令（关键字）。以下是一些常用的 `Dockerfile` 关键字：
1. `FROM`：指定基础镜像，用于构建当前镜像的起点。
2. `RUN`：在镜像中执行命令，用于安装软件包、运行脚本等。
3. `COPY`：将文件或目录从构建环境复制到镜像中指定的路径。
4. `ADD`：类似于 `COPY`，但是支持更多的功能，如自动解压缩压缩文件。
5. `WORKDIR`：设置容器中的工作目录。
6. `ENV`：设置环境变量。
7. `EXPOSE`：声明容器运行时监听的端口号。
8. `CMD`：设置容器启动时的默认命令。
9. `ENTRYPOINT`：配置容器启动时执行的命令，与 `CMD` 类似，但可以与附加的命令参数进行组合。
10. `VOLUME`：声明持久化数据的挂载点。
11. `LABEL`：为镜像添加元数据，如作者、版本等信息。
12. `ARG`：在构建镜像时传递参数。
13. `ONBUILD`：定义在当前镜像作为基础镜像时触发的操作。
14. `HEALTHCHECK`：设置容器的健康检查。
15. `USER`：指定容器中运行的用户。

这些关键字组合在一起，可以定义一个完整的 `Dockerfile`，描述了如何构建和配置一个 `Docker` 镜像。