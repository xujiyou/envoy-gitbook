# 在 Linux 发行版中部署 Envoy

先在 [https://github.com/tetratelabs/getenvoy/releases](https://github.com/tetratelabs/getenvoy/releases) 中下载 getenvoy 的二进制文件，当前最新版为 v0.1.8 ，下载安装步骤如下：

```bash
$ wget https://github.com/tetratelabs/getenvoy/releases/download/v0.1.8/getenvoy_0.1.8_Linux_x86_64.tar.gz
$ tar zxvf getenvoy_0.1.8_Linux_x86_64.tar.gz
$ sudo mv getenvoy /usr/bin/
```

通过 getenvoy 来运行 Envoy：

```bash
$ getenvoy run standard:1.14.1 -- --config-path ./envoy-config.yaml 
```

第一次运行较慢，需要下载 Envoy 的二进制文件，并且需要科学上网。

也可以直接安装 Envoy 的二进制文件，安装方法如下：

```bash
$ sudo yum-config-manager --add-repo https://getenvoy.io/linux/centos/tetrate-getenvoy.repo
$ sudo yum install -y getenvoy-envoy
$ envoy --version
```

在本机直接运行 Envoy：

```bash
$ envoy --config-path /etc/envoy/envoy.yaml
```

使用 getenvoy 的好处是可以

