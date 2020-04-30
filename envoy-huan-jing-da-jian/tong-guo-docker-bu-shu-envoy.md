# 通过 Docker 部署 Envoy

Envoy 官方并没有二进制文件（但可以自己编译），只能使用 Envoy 提供的官方镜像为基础镜像，然后在这个基础镜像的基础上来添加自己的服务。

Envoy 分别提供了几种镜像供用户使用，这些镜像分别是：

* envoyproxy/envoy
* envoyproxy/envoy-alpine
* envoyproxy/envoy-alpine-debug
* envoyproxy/envoy-dev
* envoyproxy/envoy-alpine-dev
* envoyproxy/envoy-alpine-debug-dev

这些镜像均可以在 DockerHub 中找到，它们都是在 Ubuntu 镜像的基础上构建的，尽管使用的是 Ubuntu 镜像，但是可以通过 Docker 环境，运行在任意 Linux 发行版之上。

由于我们是为了学习使用 Envoy，所以这里选择使用  **envoyproxy/envoy:latest** 这个最新稳定版的 Envoy 镜像。

下面来通过 Docker 部署一个 Envoy。

先来写一个 Envoy 的配置文件，命名为 envoy-config.yaml ，内容如下：

```yaml
admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address:
      address: 0.0.0.0
      port_value: 20001

static_resources:
  listeners:
    - name: listener-1
      address:
        socket_address:
          address: 0.0.0.0
          port_value: 21001
      filter_chains:
        - filters:
          - name: envoy.filters.network.http_connection_manager
            typed_config:
              "@type": type.googleapis.com/envoy.config.filter.network.http_connection_manager.v2.HttpConnectionManager
              stat_prefix: ingress_http
              codec_type: AUTO
              access_log:
                name: envoy.file_access_log
                typed_config:
                  "@type": type.googleapis.com/envoy.config.accesslog.v2.FileAccessLog
                  path: /dev/stdout
              route_config:
                name: local_route
                virtual_hosts:
                  - name: local_service
                    domains:
                      - "*"
                    routes:
                      - match:
                          prefix: "/"
                        route:
                          host_rewrite: www.qq.com
                          cluster: service-qq
              http_filters:
                - name: envoy.filters.http.router

  clusters:
    - name: service-qq
      connect_timeout: 5s
      type: STRICT_DNS
      dns_lookup_family: V4_ONLY
      lb_policy: ROUND_ROBIN
      load_assignment:
        cluster_name: service-qq
        endpoints:
          - lb_endpoints:
              - endpoint:
                  address:
                    socket_address:
                      address: www.qq.com
                      port_value: 443
      transport_socket:
        name: envoy.transport_sockets.tls
        typed_config: {}
```

这个配置文件的意思就是把流量重定向到 www.qq.com ，具体配置细节会在后面的章节详细介绍。

然后编写 Dockerfile 文件，内容如下：

```text
FROM envoyproxy/envoy:latest
COPY envoy-config.yaml /etc/envoy/envoy.yaml
```

**envoyproxy/envoy:latest** 这个基础镜像会自动使用 **/etc/envoy/envoy.yaml** 作为配置文件启动 Envoy，通过查看 DockerHub 上镜像的构建历史可以发现这一点：

![](../.gitbook/assets/image%20%281%29.png)

最终的启动命令就是 `envoy -c /etc/envoy/envoy.yaml` 。

然后构建 Docker 镜像：

```text
$ docker build -t envoy:v1 .
```

运行容器：

```text
$ docker run --rm --name envoy -p 20001:20001 -p 21001:21001 envoy:v1
```

在另一个命令行测试访问，这里访问宿主机的 21001 端口，就会连接到容器的 21001 端口，Envoy 通过监听容器的 21001 端口来提供服务：

```text
$ curl -IL -w "%{http_code}" -o /dev/null http://localhost:21001
```

访问之后会发现返回了 200，说明代理成功了，查看容器的日志，发现了一条 HTTP 状态码为 200 的日志，说明 Envoy 成功对流量进行了代理。

除了提供代理服务之外，Envoy 还对外提供了一个 Admin 管理端口，这里为 20001，浏览器访问这个端口会发现如下界面：

![](../.gitbook/assets/image%20%282%29.png)

这个 Admin 管理端对我们学习调试 Envoy 有很大的帮助，在本章的最后一个小节会对 Admin 管理端进行详细讲解。

通过以上过程，就理解了 Envoy 的基础镜像是如何运行的，但 Envoy 绝大多数情况下都不会单独运行，需要配合其他服务一起运行，但是一个容器内的服务是有限的，这时候就需要在 Docker-Compose 或 Kubernetes 上运行 Envoy 了。

