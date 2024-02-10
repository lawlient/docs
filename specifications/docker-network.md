# docker容器网络规划

## 背景

默认容器网络为**docker_container_name**_default，基于docker compose管理容器过程中，随着容器数量的增长，网络也越来越多。

前端通过nginx代理，起到网关作用。每次增加新容器需要代理新的网络服务，运维成本高。

通过对网络进行规范化管理，降低运维成本。


## 方案

docker虚拟化网络如下：

1. test_net
2. dev_net
3. pro_net


```bash
docker network create test_net
```

```bash
docker network create dev_net
```

```bash
docker network create pro_net
```


### docker-compose.yml管理容器

示例如下：

```
services:
    demo:
        networks:
            - testnet

networks:
    testnet:
        name: test_net
        external: true
```
