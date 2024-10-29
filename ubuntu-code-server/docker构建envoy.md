## 配置镜像
vim /etc/docker/daemon.json
```
{
    "registry-mirrors": [
        "https://dockerproxy.com",
        "https://docker.mirrors.ustc.edu.cn",
        "https://docker.nju.edu.cn"
    ]
}

```
## 使用代理构建

```
IMAGE_NAME=envoyproxy/envoy-build-ubuntu http_proxy=http://192.168.100.54:7890 https_proxy=http://192.168.100.54:7890 ./ci/run_envoy_docker.sh './ci/do_ci.sh bazel.release'
81a93046060dbe5620d5b3aa92632090a9ee4da6: Pulling from envoyproxy/envoy-build-ubuntu
```

