# 通过 Kubernetes 部署 Envoy

Kubernetes 是容器编排系统的事实标准，也是整个云原生生态的基石。Envoy 从一开始就被设计为面向云原生服务网格。

本节学习 Envoy 是如何在 Kubernetes 中运用的，尽管在实际生产环境中，并不会直接在 Kubernetes 中使用 Envoy，而是通过 Istio 管理 Envoy，但是这里学习一下有利于帮助我们理解 Envoy 在集群环境的使用，甚至可以基于此，定制开发自己的服务网格技术。



