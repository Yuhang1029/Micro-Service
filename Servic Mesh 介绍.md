# Service Mesh 介绍

**参考文档**

[什么是 Service Mesh](https://zhuanlan.zhihu.com/p/61901608)

[Pattern: Service Mesh](https://philcalcado.com/2017/08/03/pattern_service_mesh.html)

[微服务架构之「 下一代微服务 Service Mesh 」](https://zhuanlan.zhihu.com/p/71065303)

---

## 服务间通信的演变

### 时代0: 想象中的方式

<img src="https://philcalcado.com/img/service-mesh/1.png" title="" alt="" data-align="center">

很显然这是一个简化的图，隐藏了实际通信中的许多细节。现实远比想象的复杂，在实际情况中，通信需要底层能够传输字节码和电子信号的物理层来完成，涉及到几个不同的层级。

&emsp;

### 时代1: 更加现实的场景

在一开始（上世纪50年代），计算机价格昂贵，数量也有限，计算机之间的连接被很好的构造与保护。但随着价格的不断降低，计算机的数目不断增多，保障连接的可靠性变得越来越重要。此外，随着流量的增多，流量控制也变得越来越重要，这些新增的需求都意味着程序员需要在原本的服务代码中添加更多的代码，构造适合自己的逻辑去实现这些新的功能和控制，服务间也就变成了下面的样子：

<img src="https://philcalcado.com/img/service-mesh/3.png" title="" alt="" data-align="center">

在TCP协议出现之前，服务需要自己处理网络通信所面临的丢包、乱序、重试等一系列流控问题，因此服务实现中，除了业务逻辑外，还夹杂着对网络传输问题的处理逻辑。

&emsp;

### 时代2: TCP 的引入

为了避免每个服务都需要自己实现一套相似的网络传输处理逻辑，TCP协议出现了，它解决了网络传输中通用的流量控制问题，将技术栈下移，从服务的实现中抽离出来，成为操作系统网络层的一部分。

<img src="https://pic2.zhimg.com/80/v2-9e6c4c6b4229b947b4efdf63de86f695_1440w.jpg" title="" alt="" data-align="center">

&emsp;

### 时代3: 微服务的出现

在TCP出现之后，机器之间的网络通信不再是一个难题。然而随着微服务的引入，分布式系统特有的通信语义又出现了，如熔断策略、负载均衡、服务发现、认证和授权、quota限制、trace 和监控等等，一开始的处理方式和最初引入 TCP 之前一样，程序员需要针对不同的服务，根据业务需求来实现一部分所需的通信语义。

<img src="https://philcalcado.com/img/service-mesh/5.png" title="" alt="" data-align="center">

例如服务发现，在单体式服务的时代，只需要通过负载均衡，DNS 以及一些大家共同遵守的约定来实现（例如，所有服务将其 HTTP 服务器绑定到端口 8080）。然而，在分布式的环境下，这一切都会变的非常复杂，相比之前可以盲目信赖 DNS，现在需要处理客户端的负载问题，不同的环境（生产环境或是开发环境）。如果说之前只需要一行代码，现在可能是大片段落来处理引入的各种极端情况。

&emsp;

### 时代4: 微服务框架的引入

为了避免每个服务都需要自己实现一套分布式系统通信的语义功能，随着技术的发展，一些面向微服务架构的开发框架出现了，如 Twitter 的 Finagle、Facebook的 Proxygen 以及Spring Cloud 等等，这些框架实现了分布式系统通信需要的各种通用语义功能：如负载均衡和服务发现等，因此一定程度上屏蔽了这些通信细节，使得开发人员使用较少的框架代码就能开发出健壮的分布式系统，避免了相同的逻辑在每一部分的服务代码中都被重写。

<img src="https://philcalcado.com/img/service-mesh/5-a.png" title="" alt="" data-align="center">

然而这种解决方法还是存在不少问题：

- 其一，虽然框架本身屏蔽了分布式系统通信的一些通用功能实现细节，但开发者却要花更多精力去掌握和管理复杂的框架本身，在实际应用中，去追踪和解决框架出现的问题也绝非易事；
- 其二，开发框架通常只支持一种或几种特定的语言，微服务的一个重要的特性就是语言无关，但那些没有框架支持的语言编写的服务，很难融入面向微服务的架构体系，想因地制宜的用多种语言实现架构体系中的不同模块也很难做到；
- 其三，框架以 lib 库的形式和服务联编，复杂项目依赖时的库版本兼容问题非常棘手，同时，框架库的升级也无法对服务透明，服务会因为和业务无关的 lib 库升级而被迫升级；

&emsp;

### 时代5: Service Mesh 的引入

由上面的叙述可以看出，最好的办法就是将分布式引入的种种特性从服务中抽离出来。现如今，人们可以利用 HTTP 开发出非常复杂精妙的应用程序而不需要考虑 TCP 式如何进行信息传递的，这一点同样可以用在微服务上面：

<img src="https://philcalcado.com/img/service-mesh/6.png" title="" alt="" data-align="center">

直接在网络通信层级中加上一层似乎并不是很容易的事情，于是人们想到了利用代理，服务并不直接和它的下游服务联系，而是所有的流量都会经过这个代理。以 Linkerd，Envoy，NginxMesh 为代表的代理模式（边车模式 Sidecar）应运而生，这就是第一代 Service Mesh，它将分布式服务的通信抽象为单独一层，在这一层中实现负载均衡、服务发现、认证授权、监控追踪、流量控制等分布式系统所需要的功能，作为一个和服务对等的代理服务，和服务部署在一起，接管服务的流量，通过代理之间的通信间接完成服务之间的通信请求，这样上边所说的三个问题也迎刃而解。

<img src="https://philcalcado.com/img/service-mesh/6-a.png" title="" alt="" data-align="center">

根据这种构想，每一个服务都会有一个陪伴的代理 Sidecar，所有服务通过 Sidecar 来沟通，从一个全局视角来看，就会是下面这个样子：

<img src="https://philcalcado.com/img/service-mesh/mesh1.png" title="" alt="" data-align="center">

> A service mesh is a dedicated infrastructure layer for handling service-to-service communication. It’s responsible for the reliable delivery of requests through the complex topology of services that comprise a modern, cloud native application. In practice, the service mesh is typically implemented as an array of lightweight network proxies that are deployed alongside application code, without the application needing to be aware.

这是 Buoyant CEO 最早对 Service Mesh 的定义。Service Mesh 看起来就像是一个由若干服务代理所组成的错综复杂的网格。

&emsp;

### 时代6: 第二代Service Mesh

第一代Service Mesh由一系列独立运行的单机代理服务构成，为了提供统一的上层运维入口，演化出了集中式的控制面板，所有的单机代理组件通过和控制面板交互进行网络拓扑策略的更新和单机数据的汇报。这就是以 Istio 为代表的第二代 Service Mesh。

<img src="https://philcalcado.com/img/service-mesh/6-b.png" title="" alt="" data-align="center">

<img src="https://philcalcado.com/img/service-mesh/mesh3.png" title="" alt="" data-align="center">

可以看出，虽然流量仍然是直接从一个服务流向另一个服务，但是控制面板知道每一个服务的存在，也为引入接入控制和埋点提供了便利。

&emsp;

## 工作流程

以第一代 Service Mesh 为例，

<img src="https://pic3.zhimg.com/80/v2-3d16bacebf115f3a72e7c5642f8eb622_1440w.jpg" title="" alt="" data-align="center">

图中可以看到，每一个服务实例的部署都会同时部署一个 Linkerd 实例，并且服务之间的调用并不是直接进行的，而是先经过 Linkerd 模块去代理转发完成的。例如：Service A 需要调用 Service B 的时候，Service A 是把请求发给与它一起部署的 Linkerd-1 的，然后这个 Linkerd-1 将请求发给 Service B 所部署模块的 Linkerd-2，然后 Linkerd-2 再将请求内容转发给 Service B 处理。因此，在整个流程中，Service A 和 Service B 只需要关注自己的业务逻辑即可，无需关注任何服务框架的功能，这些服务框架的功能都是由Linkerd 去实现，Linkerd 要做 负载均衡、熔断、限流、监控等等。

&emsp;

## 两个核心点

### SideCar

上面说到的与服务部署在一起的轻量级网络代理也就是指 SideCar，它的作用就是实现服务框架的各项功能，这样，就可以让服务（Service A 或 Service B）回归业务本质。  传统的微服务架构中，各种服务框架的功能（例如：服务发现、负载均衡、限流熔断等）都代码逻辑或多或少的都需要耦合到服务实例的代码里，给服务实例增加了很多无关业务的代码，也带来了复杂度。  有了SideCar之后，服务节点只做业务逻辑自身的功能，服务之间的调用交给了SideCar，由SideCar去注册服务、去做服务发现、去做请求路由、去实现熔断限流、去做日志统计。  

### Control Plane

那么在这种新的微服务架构中，所有的 SideCar 组成在一起，其实就是一个服务网格了。那么这个大型的服务网格并不是完全自治的，它还需要一个统一的控制节点，也就是 Control Plane。Control Plane 是用来从全局的角度上控制 SideCar 的。比如 它负责所有 SideCar 的注册，它存储一个统一的路由表，帮助各个 SideCar 进行负载均衡和请求调度。它收集所有 SideCar 的监控信息和日志数据。它就相当于 Service Mesh架构的大脑。Control Plane 控制着 SideCar 来实现服务治理的各项功能。

&emsp;

## 总结

总结一下，Service Mesh具有如下优点：

- 屏蔽分布式系统通信的复杂性(负载均衡、服务发现、认证授权、监控追踪、流量控制等等)，服务只用关注业务逻辑；
- 真正的语言无关，服务可以用任何语言编写，只需和Service Mesh通信即可；
- 对应用透明，Service Mesh组件可以单独升级；

更加通俗概括的说， Service Mesh 的出现就是不断的将通用的逻辑下沉，和业务逻辑剥离，然后再取解决因为解耦和下沉带来的性能问题。
