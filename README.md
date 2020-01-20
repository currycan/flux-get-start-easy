# 基于Flux项目的云原生GitOps实践

## 什么是GitOps?
GitOps, 这已经并不是一个新鲜的概念了。2018年5月初在丹麦举行的哥本哈根KubeCon大会上，Weaveworks公司的演讲将GitOps与Istio Service Mesh进行了集成，如果说以前Docker Swarm与Kubernetes竞争之时Docker公司提出了Docker Native，CNCF基于Kubernetes提出了自己的Cloud Native，毫不夸张的说，Weaveworks公司开源的Weave Flux也可以说是GitOps的‘Native’了。而在2019年8月20日，Flux项目也最终成功加入了CNCF Sandbox，成为了CNCF Sandbox中的一员。

![flux-cd-diagram.png](imgs/flux-cd-diagram.png?raw=true)

当然，GitOps的概念是从DevOps慢慢延伸出来的。把时间轴向前调调整，如2014年左右如火如荼的DevOps一样，当时从大到小的互联网企业都在招聘DevOps工程师。然而慢慢脱离了以前DevOps理念的不成熟，随着DevOps的发展，人们才慢慢意识到DevOps并不是所谓的"运维开发", 而是一种注重了开发团队、测试团队、运维团队的更好的沟通协作，从而去实现持续集成和持续交付的最佳实践。

如果说之前对DevOps的理念理解是"顾名思义"而导致的问题，那么现在的GitOps也多多少少面临着同样的境地，GitOps绝非是仅仅用Git去做CI/CD的Pipeline，既然Weaveworks开源的Weave Flux可以成为GitOps的主流实践，其给出的描述是这样的: “如果说DevOps的CI/CD Pipeline的终点是互联网公司交付的产品或者是我们最终发布的线上业务，GitOps则把目标转向了当前的容器编排事实标准--Kubernetes，而GitOps则是一种进行Kubernetes集群管理和应用程序交付的方法。”

这样一来，GitOps就于传统的DevOps划清了界限。更明确一点说: “DevOps注重的是产品发布中开发/运维/测试的沟通与协作，GitOps则更加贴近集群管理。这个集群还得是"拥抱云原生"基础设施的Kubernetes集群。”

既然贴近了云原生和Kubernetes，就不得不提到云原生12要素。更值得关注的是，这12要素的第一条就是"基准代码，多份部署"。GitOps的设计者也意识到了这一点，在GitOps中，一旦Git仓库中的代码被更改，CI/CD Pipeline也就会对我们的Kubernetes集群进行更改。GitOps摒弃了传统部署环境的多份环境多份配置文件，并且设计者也应用了Kubernetes控制循环的思想，用Git管理的Kubernetes集群的期望状态也会和Git仓库中的实时状态不断地进行比较。

接下来的实战就一起来看看Flux项目是怎么用Git来管理整个Kubernetes集群的。
***

## Flux CD 实践

Flux的安装默认提供了两种方式，传统的使用yaml文件部署的方式或者通过Helm的部署。但是Flux暂未支持Helm的V3版本，所以我们使用传统方式去部署(笔者闲谈:由于笔者使用的是Helm V3版本，也坚信去Tiller化的正确性，但是目前开源社区对Helm V3版本的响应程度着实不高，也导致更多人把精力转向了Kubernetes原生的Kustomize)

我们直接Clone Flux项目的Github Repo

```shell
% git clone https://github.com/fluxcd/flux
% cd flux/
% vim deploy/flux-deployment.yaml
```

在这里，我们需要将--git-url更改为存储生产环境yaml文件的Github Repo，当然如果不想把生产环境的yaml文件托管在Github上，Flux也提供了Gitlab的支持去更好的进行私有环境的部署与管理。

![Git-Repo-Config.png](imgs/Git-Repo-Config.png?raw=true)

```shell
--git-url=git@github.com:YOUR-GITHUB/REPO-NAME
```# 基于Flux项目的云原生GitOps实践

## 什么是GitOps?
GitOps, 这已经并不是一个新鲜的概念了。2018年5月初在丹麦举行的哥本哈根KubeCon大会上，Weaveworks公司的演讲将GitOps与Istio Service Mesh进行了集成，如果说以前Docker Swarm与Kubernetes竞争之时Docker公司提出了Docker Native，CNCF基于Kubernetes提出了自己的Cloud Native，毫不夸张的说，Weaveworks公司开源的Weave Flux也可以说是GitOps的‘Native’了。而在2019年8月20日，Flux项目也最终成功加入了CNCF Sandbox，成为了CNCF Sandbox中的一员。
![](media/15794183193667/15794273849292.jpg)
当然，GitOps的概念是从DevOps慢慢延伸出来的。把时间轴向前调调整，如2014年左右如火如荼的DevOps一样，当时从大到小的互联网企业都在招聘DevOps工程师。然而慢慢脱离了以前DevOps理念的不成熟，随着DevOps的发展，人们才慢慢意识到DevOps并不是所谓的"运维开发", 而是一种注重了开发团队、测试团队、运维团队的更好的沟通协作，从而去实现持续集成和持续交付的最佳实践。
如果说之前对DevOps的理念理解是"顾名思义"而导致的问题，那么现在的GitOps也多多少少面临着同样的境地，GitOps绝非是仅仅用Git去做CI/CD的Pipeline，既然Weaveworks开源的Weave Flux可以成为GitOps的主流实践，其给出的描述是这样的: “如果说DevOps的CI/CD Pipeline的终点是互联网公司交付的产品或者是我们最终发布的线上业务，GitOps则把目标转向了当前的容器编排事实标准--Kubernetes，而GitOps则是一种进行Kubernetes集群管理和应用程序交付的方法。”

这样一来，GitOps就于传统的DevOps划清了界限。更明确一点说: “DevOps注重的是产品发布中开发/运维/测试的沟通与协作，GitOps则更加贴近集群管理。这个集群还得是"拥抱云原生"基础设施的Kubernetes集群。”

既然贴近了云原生和Kubernetes，就不得不提到云原生12要素。更值得关注的是，这12要素的第一条就是"基准代码，多份部署"。GitOps的设计者也意识到了这一点，在GitOps中，一旦Git仓库中的代码被更改，CI/CD Pipeline也就会对我们的Kubernetes集群进行更改。GitOps摒弃了传统部署环境的多份环境多份配置文件，并且设计者也应用了Kubernetes控制循环的思想，用Git管理的Kubernetes集群的期望状态也会和Git仓库中的实时状态不断地进行比较。

接下来的实战就一起来看看Flux项目是怎么用Git来管理整个Kubernetes集群的。
## Flux CD 实践demo
Clone[ Flux项目](https://github.com/fluxcd/flux)的Github Repo

```
$ git clone https://github.com/fluxcd/flux
$ cd flux/
$ vim deploy/flux-deployment.yaml
```
在这里，我们需要将--git-url更改为存储生产环境yaml文件的Github Repo，当然如果不想把生产环境的yaml文件托管在Github上，Flux也提供了Gitlab的支持去更好的进行私有环境的部署与管理。将`- --git-path=subdir1,subdir2`修改为`- --git-path=namespaces,workloads`（可选，根据需要选择）


```
142         # Replace the following URL to change the Git repository used by Flux.
143         # HTTP basic auth credentials can be supplied using environment variables:
144         # https://$(GIT_AUTHUSER):$(GIT_AUTHKEY)@github.com/user/repository.git
145         - --git-url=git@github.com:currycan/flux-get-started-easy
146         - --git-branch=master
147         # Include this if you want to restrict the manifests considered by flux
148         # to those under the following relative paths in the git repository
149         # - --git-path=namespaces,workloads
150         - --git-label=flux-sync
151         - --git-user=Flux automation
152         - --git-email=flux@example.com
```
部署Flux到Kubernetes集群中

```
$ kubectl apply -f deploy
```
等待服务全都启动完成

```
$ kubectl get po -n flux  -w
NAME                         READY   STATUS    RESTARTS   AGE
flux-bd8857874-4xdnr         1/1     Running   0          85s
memcached-86bdf9f56b-kcjzh   1/1     Running   0          85s
```
安装fluxctl

```
$ wget https://github.com/fluxcd/flux/releases/download/1.17.1/fluxctl_linux_amd64
$ chmod 770 fluxctl_linux_amd64 && mv fluxctl_linux_amd64 /usr/local/bin/fluxctl
```
fluxctl安装好之后，获取flux自动生成的访问ssh的公钥。然后配置公钥到Github上，如何配置可以参看[使用ssh连接github](https://blog.csdn.net/Zhangxichao100/article/details/78702385)
```
$ fluxctl identity --k8s-fwd-ns flux
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCrmVAZY6gQithxZ0mGwPBjej3ySYMiH1v84UO1WlH2VkP1hILGWcdQkYIfgoK2NiCmufOmZAyvoivfN12T9s3jtf7SpZe5ylZQaGeYeFBnZI2DvAdvtdgMFz493aya7cEeBacWCIg0pYUmDuWlIYjByGgNRYTpdmhpODTOnTQG843g9Y1MSOuRlvkdIjEYnLbYTLlYZ6NK3lJuap/IQFPMghUVi+r8gkku1MtKNkMV9ZXhgQfExgiW++WVtMHkREIO82tcsHA4m+0mXkwChLOJitrn/3TGMqucM5lWyI8bYVssjTsjuEwWvFD2FdpD7CkYL71FbLvddMQWMA8E5+wG0CSSErPdidHTVoTPWXHIKjkJOOdl+sAG31RY/LIezztYm1jmQATuZ3pSBS/DERvxtPiffr6nY3C679OYlPrwHGlV20HdnxJFQzMuOZ77AvHgEE2DIc0wxVCkY/46DeFaEwpnEDBGZvjatMYwky+xIttxB7wfxyc+ETzEOemxap0= root@flux-564cccff89-qmwk6
```
显然这种方式不算特别友好，如果我们想使用自己已经创建使用的公钥，从而更加方便的管理，也是可行的。具体的做法是先删除原本Flux的secret，再通过手动创建secret即可。注意：手动创建的私钥一定名称要为`generic flux-git-deploy`：
```
$ kubectl create secret -n <flux_ns>  generic flux-git-deploy --from-file=identity=</path/to/private_key>
```
举例来说，获取原有配置在github上的ssh密钥(**私钥**)，这里文件名为`git_ssh_key`，生成flux ssh secret：

```
$ kubectl create secret -n flux generic flux-git-deploy --from-file=identity=./git_ssh_key
Error from server (AlreadyExists): secrets "flux-git-deploy" already exists
```
这里会报错，secret已存在，需先删除

```
$ kubectl delete secrets -n flux flux-git-deploy
secret "flux-git-deploy" deleted
```
删除secret后，flux如果报错，请仔细确认证书是否正确

验证是否成功：

```
$ kubectl create secret -n flux generic flux-git-deploy --from-file=identity=./git_ssh_key
secret/flux-git-deploy created
$ fluxctl sync --k8s-fwd-ns flux
```

部署demo

```
$ fluxctl sync --k8s-fwd-ns flux
Synchronizing with ssh://git@github.com/currycan/flux-get-start-easy
Revision of master to apply is 9e44ff2
Waiting for 9e44ff2 to be applied ...
Done.
```
查看是否部署完成
```
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

```
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
```
$ vim nginx-deployment-flux.yaml
$ git add nginx-deployment-flux.yaml
$ git commit -m "change the version 1.16"
```
开始同步部署

```
$ fluxctl sync --k8s-fwd-ns flux
Synchronizing with ssh://git@github.com/currycan/flux-get-start-easy
Revision of master to apply is 895d133
Waiting for 895d133 to be applied ...
Done.
```
查看nginx版本

```
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

```
$ fluxctl sync --k8s-fwd-ns flux
Error: git repository ssh://git@github.com/currycan/flux-get-start-easy.git is not ready to sync (status: new)
Run 'fluxctl sync --help' for usage.
```
status new 报错，这种情况下，
* 看是否github上证书已添加
* 登陆到容器内，查看详情：

```
$ kubectl exec -it -n flux flux-fdc759885-hl5rc bash
$ cat ~/.ssh/known_hosts
$ ssh -T git@github.com
```
```
$ fluxctl sync --k8s-fwd-ns flux
Error: git repository ssh://git@github.com/currycan/flux-get-start-easy is not ready to sync (status: cloned)
Run 'fluxctl sync --help' for usage.
```

在同步git配置的时候，出现过 status cloned 报错，始终找不到原因。后来发现是部署flux的时候`flux-deployment.yaml`配置不对，确保 `--git-url`正确（出现git 地址拼写错误）。



PS: 因为官方的例子过于繁琐，笔者在这里提供了更直观的例子。这个例子很简单，只由一个Nginx-Deployment和带有NodePort的Nginx-Service组成，项目已经存储在[Github](https://github.com/youngercloud/flux-get-start-easy)

部署Flux到Kubernetes集群中

```shell
% kubectl apply -f deploy
```

PS: 如果使用minikube进行实验，请确保安装socat

```shell
% yum install -y socat
```

确保Flux Pod进入Running状态并Ready后，我们还需要下载fluxctl二进制命令包，fluxctl将与Kubernetes集群中的flux Pod进行交互。

![Flux-Deployed.png](imgs/Flux-Deployed.png?raw=true)

```shell
% wget https://github.com/fluxcd/flux/releases/download/1.13.1/fluxctl_linux_amd64
% mv fluxctl_linux_amd64 fluxctl && chmod +x fluxctl && cp fluxctl /usr/local/bin/
```

fluxctl安装好之后，我们需要部署我们的Deploy Key到Github Repo上，以实现本地集群和远端Github Repo的连调。


我们可以通过fluxctl identity命令获取Flux的SSH公钥，如果你想打造一个更方便管理的环境，Flux也可以使用系统SSH所产生的私钥，具体的做法是先删除原本Flux的secret，再通过--from-file=priveate_key的方式重新创建需要被Pod挂载的Secret私钥。

```shell
% fluxctl identity
```

当我们在Git Repo中通过SSH私密部署好Deploy Key并Allow Read/Write Access权限后，我们就可以尝试本地环境和远端Repo的同步了。

```shell
% fluxctl sync
```

如果没有问题的话，Flux会返回以下信息:

![First-Sync-Success.png](imgs/First-Sync-Success.png?raw=true)

这条信息的出现表明了集群同步已经完毕。接下来我们就可以尝试使用Git去管理Kubernetes集群了。
 
我们先执行`kubectl get all`查看Kubernetes集群的当前状态。
可以清楚的看到，我们没有手动的使用kubectl执行任何操作，Flux已经自动的帮我们做好了本地集群和远端Git Repo的同步工作，Nginx-Pod已经处在了Running状态。

![Nginx-Deployment-Succeed.png](imgs/Nginx-Deployment-Succeed.png?raw=true)

我们可以使用curl命令对已经被Flux部署在Kubernetes集群中的Nginx进行访问测试，可以看到访问测试是成功的。

![Nginx-Old-Curl.png](imgs/Nginx-Old-Curl.png?raw=true) 	

在这时，如果我们尝试用git去对集群做出更改，整体的流程和我们平时修改代码的流程是大致相同的

**1. 如果本地仓库没有yaml文件，我们需要先从远端仓库pull下来我们的yaml**

**2. 在本地仓库去更改我们的yaml文件，在这个例子中，我对之前所部署的Nginx版本做出了修改，从1.13.12升级到了1.14.2**

![Change-to-1.14.2.png](imgs/Change-to-1.14.2.png?raw=true)

**3. git add FILE_CHANGED**

**4. git commit -m "DESCRIPTION"**

**5. git push**

一旦我们的代码被提交到远端的GitRepo仓库后，我们就可以再次使用`fluxctl sync`命令去进行同步。如果不执行这条命令，Flux也会过几分钟之后自动的去进行同步。

通过curl命令的测试结果我们可以看到，集群更新已经完毕，我们Kubernetes集群中的Nginx也修改到了1.14.2版本。

![New-Version-Success.png](imgs/New-Version-Success.png?raw=true)

##  参考连接:
* Flux项目Github Repo： https://github.com/fluxcd/flux
* Flux项目官方网站： https://fluxcd.io/


![Flux-Logo.png](imgs/Flux-Logo.png?raw=true)
