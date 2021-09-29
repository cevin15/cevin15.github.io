# Kubernetes 与 Docker 使用经验

_2021-09-29_ _13:43_

## 使用docker login 登录私有仓库

```
docker login --username=${用户名} ${私有仓库地址}
```
Docker会将token存储在`~/.docker/config.json`文件中，从而作为拉取私有镜像的凭证。

## 推送docker镜像到私有仓库
先打上私有仓库的标签
```
docker tag demo:0.0.1 ${私有仓库地址}/demo:0.0.1
```
推送
```
docker push ${私有仓库地址}/demo:0.0.1
```

## kubernetes 拉取私有仓库镜像
生成私有仓库secret
```
kubectl create secret docker-registry ${secret名} --docker-server=${私有仓库地址} --docker-username=${用户名} --docker-password=${用户密码} --docker-email=${用户邮箱}
```
yaml 文件拉取镜像时使用secret
```
    ## 配置如下
    spec:
      containers:
      - name: demo
        image: xxx/demo:0.0.1
        ports:
        - containerPort: 8015
      imagePullSecrets: # 使用secret 拉取私有仓库数据
      - name: ${secret名} # 上述步骤生成的secret名
```

## 进入容器内部
```
kubectl exec -it ${pod name} -- /bin/bash
```

## kubernetes 的kubeconfig 文件
kubeconfig 保存为~/.kube/config ，作为默认文件。指定某个config文件语法为 --kubeconfig=${config 文件路径}