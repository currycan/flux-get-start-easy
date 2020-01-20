# 基于Flux项目的云原生GitOps实践

## 什么是GitOps

GitOps, 这已经并不是一个新鲜的概念了。2018年5月初在丹麦举行的哥本哈根KubeCon大会上，Weaveworks公司的演讲将GitOps与Istio Service Mesh进行了集成，如果说以前Docker Swarm与Kubernetes竞争之时Docker公司提出了Docker Native，CNCF基于Kubernetes提出了自己的Cloud Native，毫不夸张的说，Weaveworks公司开源的Weave Flux也可以说是GitOps的‘Native’了。而在2019年8月20日，Flux项目也最终成功加入了CNCF Sandbox，成为了CNCF Sandbox中的一员。
![flux-cd-diagram.png](imgs/flux-cd-diagram.png?raw=true)
当然，GitOps的概念是从DevOps慢慢延伸出来的。把时间轴向前调调整，如2014年左右如火如荼的DevOps一样，当时从大到小的互联网企业都在招聘DevOps工程师。然而慢慢脱离了以前DevOps理念的不成熟，随着DevOps的发展，人们才慢慢意识到DevOps并不是所谓的"运维开发", 而是一种注重了开发团队、测试团队、运维团队的更好的沟通协作，从而去实现持续集成和持续交付的最佳实践。
如果说之前对DevOps的理念理解是"顾名思义"而导致的问题，那么现在的GitOps也多多少少面临着同样的境地，GitOps绝非是仅仅用Git去做CI/CD的Pipeline，既然Weaveworks开源的Weave Flux可以成为GitOps的主流实践，其给出的描述是这样的: “如果说DevOps的CI/CD Pipeline的终点是互联网公司交付的产品或者是我们最终发布的线上业务，GitOps则把目标转向了当前的容器编排事实标准--Kubernetes，而GitOps则是一种进行Kubernetes集群管理和应用程序交付的方法。”

这样一来，GitOps就于传统的DevOps划清了界限。更明确一点说: “DevOps注重的是产品发布中开发/运维/测试的沟通与协作，GitOps则更加贴近集群管理。这个集群还得是"拥抱云原生"基础设施的Kubernetes集群。”

既然贴近了云原生和Kubernetes，就不得不提到云原生12要素。更值得关注的是，这12要素的第一条就是"基准代码，多份部署"。GitOps的设计者也意识到了这一点，在GitOps中，一旦Git仓库中的代码被更改，CI/CD Pipeline也就会对我们的Kubernetes集群进行更改。GitOps摒弃了传统部署环境的多份环境多份配置文件，并且设计者也应用了Kubernetes控制循环的思想，用Git管理的Kubernetes集群的期望状态也会和Git仓库中的实时状态不断地进行比较。

接下来的实战就一起来看看Flux项目是怎么用Git来管理整个Kubernetes集群的。

## Flux CD 实践demo

部署demo应用

```bash
git clone https://github.com/currycan/flux-get-start-easy
cd flux-get-start-easy/
kubectl apply -f nginx-deployment-flux.yaml
kubectl apply -f nginx-service-flux.yaml
```

Clone[ Flux项目](https://github.com/fluxcd/flux)的Github Repo

```bash
git clone https://github.com/fluxcd/flux
cd flux/
vim deploy/flux-deployment.yaml
```

[服务端各配置项参数](https://docs.fluxcd.io/en/1.17.1/references/daemon.html)
需要修改以下字段：

* `git-url`，修改为部署文件github地址
* `git-branch`，默认为master，根据需要可修改
* `git-path`，建议设为‘.’（默认值会报错）
* `git-user`，修改为github用户名
* `git-email`，修改为GitHub邮箱

```text
143         # Replace the following URL to change the Git repository used by Flux.
144         # HTTP basic auth credentials can be supplied using environment variables:
145         # https://$(GIT_AUTHUSER):$(GIT_AUTHKEY)@github.com/user/repository.git
146         - --git-url=git@github.com:currycan/flux-get-start-easy
147         - --git-branch=master
148         # Include this if you want to restrict the manifests considered by flux
149         # to those under the following relative paths in the git repository
150         # - --git-path=subdir1,subdir2
151         - --git-path=.
152         - --git-label=flux-sync
153         - --git-user=currycan
154         - --git-email=ansandyzzt@gmail.com
```        - --git-email=flux@example.com
```

部署Flux到Kubernetes集群中

```bash
kubectl apply -f deploy
```

等待服务全都启动完成

```bash
$ kubectl get po -n flux  -w
NAME                         READY   STATUS    RESTARTS   AGE
flux-bd8857874-4xdnr         1/1     Running   0          85s
memcached-86bdf9f56b-kcjzh   1/1     Running   0          85s
```

安装fluxctl

```bash
wget https://github.com/fluxcd/flux/releases/download/1.17.1/fluxctl_linux_amd64
chmod 770 fluxctl_linux_amd64 && mv fluxctl_linux_amd64 /usr/local/bin/fluxctl
```

fluxctl安装好之后，获取flux自动生成的访问ssh的公钥。然后配置公钥到Github上，如何配置可以参看[使用ssh连接github](https://blog.csdn.net/Zhangxichao100/article/details/78702385)

```bash
$ fluxctl identity --k8s-fwd-ns flux
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCrmVAZY6gQithxZ0mGwPBjej3ySYMiH1v84UO1WlH2VkP1hILGWcdQkYIfgoK2NiCmufOmZAyvoivfN12T9s3jtf7SpZe5ylZQaGeYeFBnZI2DvAdvtdgMFz493aya7cEeBacWCIg0pYUmDuWlIYjByGgNRYTpdmhpODTOnTQG843g9Y1MSOuRlvkdIjEYnLbYTLlYZ6NK3lJuap/IQFPMghUVi+r8gkku1MtKNkMV9ZXhgQfExgiW++WVtMHkREIO82tcsHA4m+0mXkwChLOJitrn/3TGMqucM5lWyI8bYVssjTsjuEwWvFD2FdpD7CkYL71FbLvddMQWMA8E5+wG0CSSErPdidHTVoTPWXHIKjkJOOdl+sAG31RY/LIezztYm1jmQATuZ3pSBS/DERvxtPiffr6nY3C679OYlPrwHGlV20HdnxJFQzMuOZ77AvHgEE2DIc0wxVCkY/46DeFaEwpnEDBGZvjatMYwky+xIttxB7wfxyc+ETzEOemxap0= root@flux-564cccff89-qmwk6
```

显然这种方式不算特别友好，如果我们想使用自己已经创建使用的公钥，从而更加方便的管理，也是可行的。具体的做法是先删除原本Flux的secret，再通过手动创建secret即可。注意：手动创建的私钥一定名称要为`generic flux-git-deploy`：

```bash
kubectl create secret -n <flux_ns>  generic flux-git-deploy --from-file=identity=</path/to/private_key>
```

举例来说，获取原有配置在github上的ssh密钥(**私钥**)，这里文件名为`git_ssh_key`，生成flux ssh secret：

```bash
$ kubectl create secret -n flux generic flux-git-deploy --from-file=identity=./git_ssh_key
Error from server (AlreadyExists): secrets "flux-git-deploy" already exists
```

这里会报错，secret已存在，需先删除后创建

```bash
$ kubectl delete secrets -n flux flux-git-deploy
secret "flux-git-deploy" deleted
$ kubectl create secret -n flux generic flux-git-deploy --from-file=identity=./git_ssh_key
secret/flux-git-deploy created
$ kubectl delete $(kubectl get pod -n flux -o name -l name=flux) -n flux
pod "flux-587d4f7674-kkjl9" deleted$ fluxctl sync --k8s-fwd-ns flux
$ fluxctl sync --k8s-fwd-ns flux
Synchronizing with ssh://git@github.com/currycan/flux-get-start-easy
Revision of master to apply is 895d133
Waiting for 895d133 to be applied ...
Done.
```

**事实上**，如果开始就不打算使用flux创建的秘钥，可以先创建好ns后创建secret，然后apply其他资源。另外，如果配置没有问题。可直接使用`fluxctl sync --k8s-fwd-ns flux`部署服务，不需要提前部署。

查看现有demo信息

```bash
$ kubectl get po
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-594cc45b78-6bgsd   1/1     Running   0          1m
nginx-deployment-594cc45b78-lfgcm   1/1     Running   0          1m
nginx-deployment-594cc45b78-rjjjf   1/1     Running   0          1m

$  kubectl get svc
NAME               TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
nginx-service      NodePort       172.23.10.241   <none>          80:30948/TCP   1m
```

查看nginx版本

```bash
$ curl -I 10.177.92.1:30948
HTTP/1.1 200 OK
Server: nginx/1.17.7
Date: Mon, 20 Jan 2020 03:08:35 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 24 Dec 2019 13:07:53 GMT
Connection: keep-alive
ETag: "5e020da9-264"
Accept-Ranges: bytes
```

修改nginx版本，试验是否会同步部署，将nginx版本修改为：1.16，当前版本1.17.

```bash
vim nginx-deployment-flux.yaml
git add nginx-deployment-flux.yaml
git commit -m "change the version 1.16"
```

开始同步部署

```bash
$ fluxctl sync --k8s-fwd-ns flux
Synchronizing with ssh://git@github.com/currycan/flux-get-start-easy
Revision of master to apply is 895d133
Waiting for 895d133 to be applied ...
Done.
```

查看nginx版本

```bash
$ curl -I 10.177.92.1:30948
HTTP/1.1 200 OK
Server: nginx/1.16.1
Date: Mon, 20 Jan 2020 03:13:07 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 13 Aug 2019 10:05:00 GMT
Connection: keep-alive
ETag: "5d528b4c-264"
Accept-Ranges: bytes
```

遇到的问题：

```bash
$ fluxctl sync --k8s-fwd-ns flux
Error: git repository ssh://git@github.com/currycan/flux-get-start-easy.git is not ready to sync (status: new)
Run 'fluxctl sync --help' for usage.
```

status new 报错，这种情况下，

* 看是否github上证书已添加
* 登陆到容器内，查看详情：

```bash
kubectl exec -it -n flux flux-fdc759885-hl5rc bash
cat ~/.ssh/known_hosts
ssh -T git@github.com
```

```bash
$ fluxctl sync --k8s-fwd-ns flux
Error: git repository ssh://git@github.com/currycan/flux-get-start-easy is not ready to sync (status: cloned)
Run 'fluxctl sync --help' for usage.
```

在同步git配置的时候，出现过 status cloned 报错，始终找不到原因。后来发现是部署flux的时候`flux-deployment.yaml`配置不对，确保 `--git-url`正确（出现git 地址拼写错误）

以上确认后无误后执行：

```bash
kubectl delete $(kubectl get pod -n flux -o name -l name=flux) -n flux
```

另外，[官方常见问题答疑](https://github.com/fluxcd/flux/blob/master/docs/faq.md)
