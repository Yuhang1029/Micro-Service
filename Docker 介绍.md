# Docker 介绍

**参考文档**

[Docker 入门教程 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)

[可能是把Docker的概念讲的最清楚的一篇文章](http://dockone.io/article/6051)

[Docker — 从入门到实践](https://yeasy.gitbook.io/docker_practice/)

---

## 为什么要用 Docker

### 环境配置的难点

软件开发最大的麻烦事之一，就是环境配置。用户计算机的环境都不相同，你怎么知道自家的软件，能在那些机器跑起来？用户必须保证两件事：操作系统的设置，各种库和组件的安装。只有它们都正确，软件才能运行。举例来说，安装一个 Python 应用，计算机必须有 Python 引擎，还必须有各种依赖，可能还要配置环境变量。如果某些老旧的模块与当前环境不兼容，那就麻烦了。开发者常常会说："它在我的机器可以跑了"（It works on my machine），言下之意就是，其他机器很可能跑不了。

传统上认为，软件编码开发/测试结束后，所产出的成果即是程序或是能够编译执行的二进制字节码等 ( Java 为例)。而为了让这些程序可以顺利执行，开发团队也得准备完整的部署文件，让维运团队得以部署应用程式，开发需要清楚的告诉运维部署团队，用的全部配置文件 + 所有软件环境。不过，即便如此，仍然常常发生部署失败的状况。

&emsp;

### 容器与虚拟机

#### 虚拟机

虚拟机（virtual machine）就是带环境安装的一种解决方案。它可以在一种操作系统里面运行另一种操作系统，比如在 Windows 系统里面运行 Linux 系统。应用程序对此毫无感知，因为虚拟机看上去跟真实系统一模一样，而对于底层系统来说，虚拟机就是一个普通文件，不需要了就删掉，对其他部分毫无影响。

虽然用户可以通过虚拟机还原软件的原始环境。但是，这个方案有几个缺点：

1. 资源占用多
   
   虚拟机会独占一部分内存和硬盘空间。它运行的时候，其他程序就不能使用这些资源了。哪怕虚拟机里面的应用程序，真正使用的内存只有 1MB，虚拟机依然需要几百 MB 的内存才能运行。

2. 冗余步骤多
   
   虚拟机是完整的操作系统，一些系统级别的操作步骤，往往无法跳过，比如用户登录。

3. 启动慢
   
   启动操作系统需要多久，启动虚拟机就需要多久。可能要等几分钟，应用程序才能真正运行。

&emsp;

#### 容器

由于虚拟机存在这些缺点，Linux 发展出了另一种虚拟化技术：Linux 容器（Linux Containers，缩写为 LXC）。Linux 容器不是模拟一个完整的操作系统，而是对进程进行隔离，或者说，在正常进程的外面套了一个保护层，对于容器里面的进程来说，它接触到的各种资源都是虚拟的，从而实现与底层系统的隔离。由于容器是进程级别的，相比虚拟机有很多优势：

1. 启动快
   
   容器里面的应用，直接就是底层系统的一个进程，而不是虚拟机内部的进程。所以，启动容器相当于启动本机的一个进程，而不是启动一个操作系统，速度就快很多。

2. 资源占用少
   
   容器只占用需要的资源，不占用那些没有用到的资源；虚拟机由于是完整的操作系统，不可避免要占用所有资源。另外，多个容器可以共享资源，虚拟机都是独享资源。

3. 体积小
   
   容器只要包含用到的组件即可，而虚拟机是整个操作系统的打包，所以容器文件比虚拟机文件要小很多。

总之，容器有点像轻量级的虚拟机，能够提供虚拟化的环境，但是成本开销小得多。一句话概括，**容器就是将软件打包成标准化单元，以用于开发、交付和部署**。  

- 容器镜像是轻量的、可执行的独立软件包 ，包含软件运行所需的所有内容：代码、运行时环境、系统工具、系统库和设置。
- 容器化软件适用于基于 Linux 和 Windows 的应用，在任何环境中都能够始终如一地运行。
- 容器赋予了软件独立性，使其免受外在环境差异（例如，开发和预演环境的差异）的影响，从而有助于减少团队间在相同基础设施上运行不同软件时的冲突。

&emsp;

### Docker

**Docker 属于 Linux 容器的一种封装，提供简单易用的容器使用接口**。Docker 将应用程序与该程序的依赖，打包在一个文件里面。运行这个文件，就会生成一个虚拟容器。程序在这个虚拟容器里运行，就好像在真实的物理机上运行一样。有了 Docker，就不用担心环境问题。总体来说，Docker 的接口相当简单，用户可以方便地创建和使用容器，把自己的应用放入容器。容器还可以进行版本管理、复制、分享、修改，就像管理普通的代码一样。

Docker 的主要用途，目前有三大类：

* **提供一次性的环境**。比如，本地测试他人的软件、持续集成的时候提供单元测试和构建的环境。

* **提供弹性的云服务**。因为 Docker 容器可以随开随关，很适合动态扩容和缩容。

* **组建微服务架构**。通过多个容器，一台机器可以跑多个服务，因此在本机就可以模拟出微服务架构。

&emsp;

## Docker 三大组件

Docker 的三个基本要素是镜像 (image)，容器 (container) 和仓库 (repository)。

### 镜像

镜像是一种轻量级、可执行的独立软件包，它包含运行某个软件所需的所有内容，我们把应用程序和配置依赖打包好形成一个可交付的运行环境(包括代码、运行时需要的库、环境变量和配置文件等)，这个打包好的运行环境就是 image 镜像文件。**只有通过这个镜像文件才能生成 Docker 容器实例 (类似 Java 中 new 出来一个对象)**。

UnionFS (联合文件系统): Union 文件系统 (UnionFS) 是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下 (unite several directories into a single virtual filesystem)。Union 文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像 (没有父镜像)，可以制作各种具体的应用镜像。镜像分层最大的一个好处就是共享资源，方便复制迁移，就是为了复用。比如说有多个镜像都从相同的 base 镜像构建而来，那么 Docker Host 只需在磁盘上保存一份 base 镜像；同时内存中也只需加载一份 base 镜像，就可以为所有容器服务了。而且镜像的每一层都可以被共享。

&emsp;

### 容器

镜像（`Image`）和容器（`Container`）的关系，就像是面向对象程序设计中的 `类` 和 `实例` 一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的命名空间，因此容器可以拥有自己的 `root` 文件系统、自己的网络配置、自己的进程空间，甚至自己的用户 ID 空间。容器内的进程是运行在一个隔离的环境里，使用起来，就好像是在一个独立于宿主的系统下操作一样。

前面讲过镜像使用的是分层存储，容器也是如此。每一个容器运行时，是以镜像为基础层，在其上创建一个当前容器的存储层，我们可以称这个为容器运行时读写而准备的存储层为**容器存储层**。容器存储层的生存周期和容器一样，容器消亡时，容器存储层也随之消亡。因此，任何保存于容器存储层的信息都会随容器删除而丢失。

&emsp;

### 仓库

镜像构建完成后，可以很容易的在当前宿主机上运行，但是，如果需要在其它服务器上使用这个镜像，我们就需要一个集中的存储、分发镜像的服务，Docker Registry 就是这样的服务。一个 Docker Registry 中可以包含多个 **仓库**（`Repository`）；每个仓库可以包含多个 **标签**（`Tag`）；每个标签对应一个镜像。通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 `<仓库名>:<标签>` 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 `latest` 作为默认标签。仓库分为公有仓库和私有仓库。公有仓库是开放给用户使用、允许用户管理镜像的 Registry 服务。一般这类公开服务允许用户免费上传、下载公开的镜像，并可能提供收费服务供用户管理私有镜像。除了使用公有仓库外，用户还可以在本地搭建私有仓库。

&emsp;

## 使用镜像

### 获取镜像

Docker 运行容器前需要本地存在对应的镜像，如果本地不存在该镜像，Docker 会从镜像仓库下载该镜像。

从 Docker 镜像仓库获取镜像的命令是 `docker pull`。其命令格式为：

`docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]`

具体的选项可以通过 `docker pull --help` 命令看到，这里我们说一下镜像名称的格式。

- Docker 镜像仓库地址：地址的格式一般是 `<域名/IP>[:端口号]`。默认地址是 Docker Hub(`docker.io`)。

- 仓库名：如之前所说，这里的仓库名是两段式名称，即 `<用户名>/<软件名>`。对于 Docker Hub，如果不给出用户名，则默认为 `library`，也就是官方镜像。
  
  &emsp;

### 运行镜像

有了镜像后，我们就能够以这个镜像为基础启动并运行一个容器。指令如下：

`docker run [OPTIONS] image_name`

`--name` : 为容器指定一个名称

`-d` : 后台运行容器并返回容器 Id ，即**启动守护式容器**

`-it` : 以交互模式运行容器，并且为容器分配一个伪输入终端，后面可以加上 `/bin/bash`，代表通过 bash 来交互

`-p hostPort:containerPort` : 指定端口映射，`hostPort` 代表着希望访问的主机的端口号，找到 docker，`containerPort` 代表着访问 docker 内部哪一个容器。

&emsp;

### 列出镜像

`docker image ls` 列出主机上的镜像。

该指令执行完后会出现一个列表，具体含义如下：

| Repository | Tag      | Image ID | Created | Virtual Size |
| ---------- | -------- | -------- | ------- | ------------ |
| 镜像的仓库源     | 镜像的标签版本号 | 对应ID     | 创建时间    | 大小           |

同一个仓库源可以有不同的 Tag，代表不同的版本，使用 `Repository : Tag` 来区分镜像。如果不指定具体的版本标签默认使用最新版本。

`docker search 镜像名`

查询某一个镜像是否存在。

&emsp;

### 删除本地镜像

当你要一次删除多张镜像时，可以使用一种方法。首先只需列出镜像即可获取镜像的 ID，然后执行简单的命令：

`docker rmi <your-image-id> <your-image-id> ...`

列出镜像的 ID，每个 ID 之间留一个空

&emsp;

### Commit 指令

`docker commit container_id created_image_name:tag` 当我们运行一个容器的时候（如果不使用卷的话），我们做的任何文件修改都会被记录于容器存储层里。而 Docker 提供了一个 `docker commit` 命令，可以将容器的存储层保存下来成为镜像。换句话说，就是在原有镜像的基础上，再叠加上容器的存储层，并构成新的镜像。以后我们运行这个新镜像的时候，就会拥有原有容器最后的文件变化。

使用 `docker commit` 命令虽然可以比较直观的帮助理解镜像分层存储的概念，但是实际环境中并不会这样使用。使用 `docker commit` 意味着所有对镜像的操作都是黑箱操作，生成的镜像也被称为 **黑箱镜像**，换句话说，就是除了制作镜像的人知道执行过什么命令、怎么生成的镜像，别人根本无从得知。而且，即使是这个制作镜像的人，过一段时间后也无法记清具体的操作。这种黑箱镜像的维护工作是非常痛苦的。

&emsp;

### 利用 Dockerfile 构建镜像

Dockerfile 是一个文本文件，其内包含了一条条的**指令(Instruction)**，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。构建的三个步骤包括：

- 编写 Dockerfile 文件

- `docker build` 命令构建镜像

- `docker run` 根据镜像运行容器实例

例如下面这个例子：

```bash
$ mkdir mynginx
$ cd mynginx
$ touch Dockerfile
```

```dockerfile
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

从应用软件的角度来看，Dockerfile、Docker 镜像与 Docker 容器分别代表软件的三个不同阶段，

-  Dockerfile 是软件的原材料

-  Docker 镜像是软件的交付品

-  Docker 容器则可以认为是软件镜像的运行态，也即依照镜像运行的容器实例

Dockerfile 面向开发，Docker 镜像成为交付标准，Docker 容器则涉及部署与运维，三者缺一不可，合力充当 Docker 体系的基石。

&emsp;

#### Dockerfile 常用保留字段

`FROM`：基本都出现在第一行，代表该镜像基于哪一个镜像。

`RUN`：RUN + 命令行命令，等同于直接在终端输入 SHELL 指令。

Dockerfile 中每一个指令都会建立一层，`RUN` 也不例外。每一个 `RUN` 的行为，就和刚才我们手工建立镜像的过程一样：新建立一层，在其上执行这些命令，执行结束后，`commit` 这一层的修改，构成新的镜像。

`WORKDIR`：在容器创建后，指定终端默认登录到的工作根目录。

`ENV`：用来在构建镜像过程中设置环境变量。

`VOLUME`：容器数据卷，用于保存和持久化工作。

`ADD`：将宿主机目录下的文件拷贝进镜像且会自动处理 URL 和解压 tar 压缩包。

`COPY`：类似 `ADD`，拷贝文件和目录到镜像中。`COPY src dest`。

`CMD`：指定容器启动后的要干的事情。

```dockerfile
FROM centos

ENV MYPATH /usr/local
WORKDIR $MYPATH

#安装 vim 编辑器
RUN yum -y install vim
#安装 ifconfig 命令查看网络IP
RUN yum -y install net-tools
#安装 java8 及 lib 库
RUN yum -y install glibc.i686
RUN mkdir /usr/local/java
#ADD 是相对路径jar,把jdk-8u171-linux-x64.tar.gz添加到容器中,安装包必须要和Dockerfile文件在同一位置
ADD jdk-8u171-linux-x64.tar.gz /usr/local/java/
#配置 java 环境变量
ENV JAVA_HOME /usr/local/java/jdk1.8.0_171
ENV JRE_HOME $JAVA_HOME/jre
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
ENV PATH $JAVA_HOME/bin:$PATH

EXPOSE 80

CMD echo $MYPATH
CMD echo "success--------------ok"
CMD /bin/bash
```

#### 构建镜像

以下面这个 Dockerfile 举例：

```dockerfile
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

在 `Dockerfile` 文件所在目录执行：

```bash
$ docker build -t nginx:v3 .
Sending build context to Docker daemon 2.048 kB
Step 1 : FROM nginx
 ---> e43d811ce2f4
Step 2 : RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
 ---> Running in 9cdc27646c7b
 ---> 44aa4490ce2c
Removing intermediate container 9cdc27646c7b
Successfully built 44aa4490ce2c
```

这里我们使用了 `docker build` 命令进行镜像构建。其格式为：

`docker build [选项] <上下文路径/URL/->`

如果注意，会看到 `docker build` 命令最后有一个 `.`。`.` 表示当前目录，而 `Dockerfile` 就在当前目录，因此不少初学者以为这个路径是在指定 `Dockerfile` 所在路径，这么理解其实是不准确的。如果对应上面的命令格式，你可能会发现，这是在指定 **上下文路径**。那么什么是上下文呢？

首先我们要理解 `docker build` 的工作原理。Docker 在运行时分为 Docker 引擎（也就是服务端守护进程）和客户端工具。Docker 的引擎提供了一组 REST API，被称为 `Docker Remote API`，而如 `docker` 命令这样的客户端工具，则是通过这组 API 与 Docker 引擎交互，从而完成各种功能。因此，虽然表面上我们好像是在本机执行各种 `docker` 功能，但实际上，一切都是使用的远程调用形式在服务端（Docker 引擎）完成。也因为这种 C/S 设计，让我们操作远程服务器的 Docker 引擎变得轻而易举。

&emsp;

## 操作容器

### 启动容器

启动容器有两种方式，一种是基于镜像新建一个容器并启动，另外一个是将在终止状态（`exited`）的容器重新启动。

前面已经讲过了 `docker run` 的指令使用方法，这里主要说一下在后台的标准操作的流程：

- 检查本地是否存在指定的镜像，不存在就从 registry 下载

- 利用镜像创建并启动一个容器

- 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层

- 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去

- 从地址池配置一个 ip 地址给容器

- 执行用户指定的应用程序

- 执行完毕后容器被终止

如果需要重新启动一个容器，可以用：

`docker start container_id` 启动已经停止的容器

`docker restart container_id` 重启容器

&emsp;

### 进入容器

某些时候需要进入容器进行操作，包括使用 `docker attach` 命令或 `docker exec` 命令，推荐大家使用 `docker exec` 命令。

对于 attach 命令，下面的示例显示了如何使用：

```bash
$ docker run -it -d ubuntu
243c32535da7d142fb0e6df616a3c3ada0b8ab417937c853a9e1c251f499f550

$ docker container ls
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
243c32535da7 ubuntu:latest "/bin/bash" 18 seconds ago Up 17 seconds nostalgic_hypatia

$ docker attach 243c
root@243c32535da7:/#
```

*注意：* 如果从这个 stdin 中 exit，会导致容器的停止。

如果使用 `exec`，指令如下：

`docker exec -it container_id bashShell`

进入正在运行的容器并以命令行交互，`exec` 是在容器中打开新的终端，并且可以启动新的进程，用 `exit` 退出，不会导致容器的停止。

&emsp;

### 查看容器

`docker ps`

列出当前正在运行的容器，如果需要列出历史上运行过的加上参数 `-a`，如果只看 id 用参数 `-q`。

`docker logs container_id`

查看容器的相应日志

&emsp;

### 导出和导入

`docker export container_id > file.tar`

`export` 导出容器的内容留作为一个 tar 归档文件。

`cat file.tar | docker import - 镜像用户/镜像名:镜像版本号`

`import` 从 tar 包中的内容创建一个新的文件系统再导入为镜像。

&emsp;

### 终止与删除容器

`docker stop container_id` 停止容器

`docker kill container_id` 强制停止容器

`docker rm container_id` 删除已经停止的容器

&emsp;

## 容器卷

容器卷的设计目的就是数据的持久化，将 Docker 容器内的数据保存进宿主机的磁盘中，完全独立于容器的生存周期，因此 Docker 不会在容器删除时删除其挂载的数据卷。它绕过 UFS，可以提供很多有用的特性：

- `数据卷` 可以在容器之间共享和重用

- 对 `数据卷` 的修改会立马生效

- 对 `数据卷` 的更新，不会影响镜像

- `数据卷` 默认会一直存在，即使容器被删

创建一个数据卷

```bash
$ docker volume create my-vol
```

查看所有的数据卷

```bash
$ docker volume ls

DRIVER              VOLUME NAME
local               my-vol
```

删除数据卷

```bash
$ docker volume rm my-vol
```

无主的数据卷可能会占据很多空间，要清理请使用以下命令

```bash
$ docker volume prune
```
