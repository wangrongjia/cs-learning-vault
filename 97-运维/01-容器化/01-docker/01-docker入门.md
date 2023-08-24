https://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html
## 01 docker是什么
linux的容器化技术：**Linux 容器不是模拟一个完整的操作系统，而是对进程进行隔离。**
目前最流行的 Linux 容器解决方案
## 02-image文件
**Docker 把应用程序及其依赖，打包在 image 文件里面** 只有通过这个文件，才能生成 Docker 容器。dockerHub就是存放image文件的公共仓库
```bash
bash
# 列出本机的所有 image 文件。
$ docker image ls

# 删除 image 文件
$ docker image rm [imageName]
```
## 03-helloworld
```bash
# 拉取镜像
docker image pull hello-world

# 运行镜像 若本地不存在 会从远程仓库自动抓取 image 文件
docker container run hello-world

# 杀掉运行的容器
docker container kill [containID]

```
## 04 -容器文件
**image 文件生成的容器实例，本身也是一个文件，称为容器文件**也就是说，一旦容器生成，就会同时存在两个文件： image 文件和容器文件。而且关闭容器并不会删除容器文件，只是容器停止运行而已。
```bash
# 列出本机正在运行的容器
$ docker container ls

# 列出本机所有容器，包括终止运行的容器
$ docker container ls --all

# 杀掉运行的容器
docker container kill [containID]

# 终止运行的容器文件，依然会占据硬盘空间 删除容器文件
docker container rm [containerID]
```
## 05-dockerfile制作自己的容器
### 1. 下载源码：
```bash
$ git clone https://github.com/ruanyf/koa-demos.git
$ cd koa-demos
```
源码大致如下：
![](Pasted%20image%2020230226222414.png)
### 2. 编写dockerfile
```bash
FROM node:8.4
COPY . /app
WORKDIR /app
RUN npm install --registry=https://registry.npm.taobao.org
EXPOSE 3000
```

-   `FROM node:8.4`：该 image 文件继承官方的 node image，冒号表示标签，这里标签是`8.4`，即8.4版本的 node。
-   `COPY . /app`：将当前目录下的所有文件（除了`.dockerignore`排除的路径），都拷贝进入 image 文件的`/app`目录。
-   `WORKDIR /app`：指定接下来的工作路径为`/app`。
-   `RUN npm install`：在`/app`目录下，运行`npm install`命令安装依赖。注意，安装后所有的依赖，都将打包进入 image 文件。
-   `EXPOSE 3000`：将容器 3000 端口暴露出来， 允许外部连接这个端口。
### 3. 制作镜像
```bash
$ docker image build -t koa-demo .
# 或者
$ docker image build -t koa-demo:0.0.1 .
```
`-t`参数用来指定 image 文件的名字，后面还可以用冒号指定标签。如果不指定，默认的标签就是`latest`。最后的那个点表示 Dockerfile 文件所在的路径，上例是当前路径，所以是一个点。
### 4. 生成容器
`docker container run`命令会从 image 文件生成容器。
```bash
$ docker container run -p 8000:3000 -it koa-demo /bin/bash
# 或者
$ docker container run -p 8000:3000 -it koa-demo:0.0.1 /bin/bash
```
上面命令的各个参数含义如下：

-   `-p`参数：容器的 3000 端口映射到本机的 8000 端口。
-   `-it`参数：容器的 Shell 映射到当前的 Shell，然后你在本机窗口输入的命令，就会传入容器。
-   `koa-demo:0.0.1`：image 文件的名字（如果有标签，还需要提供标签，默认是 latest 标签）。
-   `/bin/bash`：容器启动以后，内部第一个执行的命令。这里是启动 Bash，保证用户可以使用 Shell。

如果一切正常，运行上面的命令以后，就会返回一个命令行提示符。

```bash
root@66d80f4aaf1e:/app#
```

这表示你已经在容器里面了，返回的提示符就是容器内部的 Shell 提示符。执行下面的命令。
```bash
root@66d80f4aaf1e:/app# node demos/01.js
```
**这样容器里就启动了nodejs应用** 
如果你的dockerfile里多一个CMD命令
```bash
FROM node:8.4
COPY . /app
WORKDIR /app
RUN npm install --registry=https://registry.npm.taobao.org
EXPOSE 3000
CMD node demos/01.js
```
它表示容器启动后自动执行`node demos/01.js`。
你可能会问，`RUN`命令与`CMD`命令的区别在哪里？简单说，`RUN`命令在 image 文件的构建阶段执行，执行结果都会打包进入 image 文件；`CMD`命令则是在容器启动后执行。另外，一个 Dockerfile 可以包含多个`RUN`命令，但是只能有一个`CMD`命令。

注意，指定了`CMD`命令以后，`docker container run`命令就不能附加命令了（比如前面的`/bin/bash`），否则它会覆盖`CMD`命令。现在，启动容器可以使用下面的命令。
```bash
$ docker container run --rm -p 8000:3000 -it koa-demo:0.0.1
```

## 其他有用的命令
docker 的主要用法就是上面这些，此外还有几个命令，也非常有用。

**（1）docker container start**

前面的`docker container run`命令是新建容器，每运行一次，就会新建一个容器。同样的命令运行两次，就会生成两个一模一样的容器文件。如果希望重复使用容器，就要使用`docker container start`命令，它用来启动已经生成、已经停止运行的容器文件。

```bash

$ docker container start [containerID]
```

**（2）docker container stop**

前面的`docker container kill`命令终止容器运行，相当于向容器里面的主进程发出 SIGKILL 信号。而`docker container stop`命令也是用来终止容器运行，相当于向容器里面的主进程发出 SIGTERM 信号，然后过一段时间再发出 SIGKILL 信号。

```bash

$ docker container stop [containerID]
```

这两个信号的差别是，应用程序收到 SIGTERM 信号以后，可以自行进行收尾清理工作，但也可以不理会这个信号。如果收到 SIGKILL 信号，就会强行立即终止，那些正在进行中的操作会全部丢失。

**（3）docker container logs**

`docker container logs`命令用来查看 docker 容器的输出，即容器里面 Shell 的标准输出。如果`docker run`命令运行容器的时候，没有使用`-it`参数，就要用这个命令查看输出。

```bash

$ docker container logs [containerID]
```

**（4）docker container exec**

`docker container exec`命令用于进入一个正在运行的 docker 容器。如果`docker run`命令运行容器的时候，没有使用`-it`参数，就要用这个命令进入容器。一旦进入了容器，就可以在容器的 Shell 执行命令了。

```bash

$ docker container exec -it [containerID] /bin/bash
```

**（5）docker container cp**

`docker container cp`命令用于从正在运行的 Docker 容器里面，将文件拷贝到本机。下面是拷贝到当前目录的写法。

```bash

$ docker container cp [containID]:[/path/to/file] .
```

