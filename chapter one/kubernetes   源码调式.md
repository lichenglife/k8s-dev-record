# kubernetes 源码调试

## 本地调试

## 远程调试

 本文记录远程调式kubernetes 源码的过程，即在搭建好集群的前提下，通过本地的goland的调式工具 dlv，远程debug 服务器上的源码，实现更好的阅读kubernets源码，更好的理解其中的设计思路;本文并不是基于当前最新版本v1.20.4，是为了测试client-gov1.20.4对于其他kubernetes集群的兼容性，如果想debug 最新代码，只需修改相应的源码版本号；

#### 1. 环境准备

- centos7 搭建kubernets v1.18.5 版本集群，建议使用1master  2node，可以很好的查看调度过程
- 在master节点 安装golang环境，并创建目录 查看v1.18.5  源码可以查看相应的golang版本，要求1.13.4以上
- 在  $GOPATH/src/k8s.io/ 目录下  执行   git clone https://github.com/kubernetes/kubernetes.git -b v1.18.5
- windows本地安装golang 环境 goland工具（以此细节，不再赘述，自行查阅）

#### 2. 代码及编译准备

-  通过正确配置  goland  Deployment 将服务器上的代码  pull 到  本地环境

  ![1617094170134](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1617094170134.png)

- 修改编译脚本

     因为源码编译脚本默认关闭了调试信息，调试符号 参数，所以修改参数启用,具体编译参数请查看上一篇kubernets源码编译

  ```shell
  vim k8s.io/kubernetes/hack/lib/golang.sh
  
  #  关闭禁止调试参数  禁止符号参数  lc
  # goldflags="${GOLDFLAGS=-s -w} $(kube::version::ldflags)"
  # goasmflags="-trimpath=${KUBE_ROOT}"
  # gogcflags="${GOGCFLAGS:-} -trimpath=${KUBE_ROOT}"
  
  ```

- 重新编译k8s 源码

  ```shell
  # 编译过程时间长短取决于服务器  内存大小如果内存较小可以选择 分批编译(指定组件编译)
  # 全量编译
  GO111MODULE=off KUBE_GIT_TREE_STATE=clean KUBE_GIT_VERSION=v1.18.5 make all GOGCFLAGS="all=-N -l"+++ 
  #或者指定组件编译（只编译kube-scheduler）
  GO111MODULE=off KUBE_GIT_TREE_STATE=clean KUBE_GIT_VERSION=v1.18.5  KUBE_BUILD_PLATFORMS=linux/amd64 make WHAT=cmd/kube-scheduler  GOGCFLAGS="all=-N -l"+++ 
  
  [root@master kubernetes]# GO111MODULE=off KUBE_GIT_TREE_STATE=clean KUBE_GIT_VERSION=v1.18.5 make all GOGCFLAGS="all=-N -l"+++ 
  +++ [0329 23:21:28] Building go targets for linux/amd64:
      ./vendor/k8s.io/code-generator/cmd/deepcopy-gen
  +++ [0329 23:21:46] Building go targets for linux/amd64:
      ./vendor/k8s.io/code-generator/cmd/defaulter-gen
  +++ [0329 23:22:01] Building go targets for linux/amd64:
      ./vendor/k8s.io/code-generator/cmd/conversion-gen
  +++ [0329 23:22:21] Building go targets for linux/amd64:
      ./vendor/k8s.io/kube-openapi/cmd/openapi-gen
  +++ [0329 23:22:42] Building go targets for linux/amd64:
      ./vendor/github.com/go-bindata/go-bindata/go-bindata
  warning: ignoring symlink /home/gowork/src/k8s.io/kubernetes/_output/local/go/src/k8s.io/kubernetes
  go: warning: "k8s.io/kubernetes/vendor/github.com/go-bindata/go-bindata/..." matched no packages
  +++ [0329 23:22:44] Building go targets for linux/amd64:
      cmd/kube-proxy
      cmd/kube-apiserver
      cmd/kube-controller-manager
      cmd/kubelet
      cmd/kubeadm
      cmd/kube-scheduler
      vendor/k8s.io/apiextensions-apiserver
      cluster/gce/gci/mounter
      cmd/kubectl
      cmd/gendocs
      cmd/genkubedocs
      cmd/genman
      cmd/genyaml
      cmd/genswaggertypedocs
      cmd/linkcheck
      vendor/github.com/onsi/ginkgo/ginkgo
      test/e2e/e2e.test
      cluster/images/conformance/go-runner
      cmd/kubemark
      vendor/github.com/onsi/ginkgo/ginkgo
      test/e2e_node/e2e_node.test
      
      
      #输出结果在  _output/bin 目录下
      
      [root@master bin]# ls
  conversion-gen  deepcopy-gen  defaulter-gen  go2make  go-bindata  kube-scheduler  openapi-gen
  ```

#### 3.调试组件

###### 3.1   kube-schedule 

- 组件介绍-- kube-schdelule
- 执行流程
- 主要代码执行流程
- **调度插件的开发**



- 正确配置goland remot配置

![1617097705628](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1617097705628.png)

- 移除正在集群中正在服务的kube-schedule 组件

  ```shell
  # 从/etc/kubernetes/manifests 目录下移走kube-schedule.yaml 文件
  [root@master manifests]# mv  kube-schedule.yaml  ../
  # 确认kube-schedule组件是否不再提供服务
  [root@master manifests]# kubectl get  po -n kube-system 
  NAME                                      READY   STATUS    RESTARTS   AGE
  coredns-859cb59455-bq7fh                  1/1     Running   32         155d
  coredns-859cb59455-glqkt                  1/1     Running   32         155d
  coredns-859cb59455-nb2rt                  1/1     Running   32         155d
  etcd-master                               1/1     Running   75         261d
  kube-apiserver-master                     1/1     Running   0          149d
  kube-controller-manager-master            1/1     Running   1          7h57m
  kube-flannel-ds-amd64-6vh6v               1/1     Running   45         261d
  kube-flannel-ds-amd64-nd6nh               1/1     Running   39         160d
  kube-flannel-ds-amd64-qx87d               0/1     Pending   0          9m1s
  kube-proxy-jr2g5                          1/1     Running   43         261d
  kube-proxy-qdkh5                          1/1     Running   75         261d
  kube-proxy-slrgg                          1/1     Running   11         261d
  nfs-client-provisioner-5ff9b8fb5d-qfnj7   1/1     Running   80         61d
  ```

  

- 在服务器上开启 监听端口提供服务

```shell
[root@master kubernetes]# dlv exec _output/bin/kube-scheduler --headless -l 192.168.108.130:2345 --api-version=2  -- \
> --authentication-kubeconfig=/etc/kubernetes/scheduler.conf  \
> --authorization-kubeconfig=/etc/kubernetes/scheduler.conf   \
> --bind-address=127.0.0.1   \
> --kubeconfig=/etc/kubernetes/scheduler.conf \
> --v=4
API server listening at: 192.168.108.130:2345
```

- 在goland中开启  remote debug 

![1617101214568](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1617101214568.png)



- 删除 pod 进行测试，是否可以正常调度，并且可以调度器工作过程中的所有日志

  ```shell
  [root@node2 ~]# kubectl   delete  po  -l  name=web-operator   
  pod "web-operator-597d4c4d88-jvhx5" deleted
  pod "web-operator-597d4c4d88-vhpws" deleted
  pod "web-operator-597d4c4d88-zzlrq" deleted
  [root@node2 ~]# kubectl  get   po  
  NAME                            READY   STATUS      RESTARTS   AGE
  example-job-9p2nx               0/1     Completed   0          138d
  web-operator-597d4c4d88-5ndwn   1/1     Running     0          10m
  web-operator-597d4c4d88-9qxkz   1/1     Running     0          10m
  web-operator-597d4c4d88-rnh7g   1/1     Running     0          10m
  
  ```

  ```shell
  I0330 01:03:52.402763     820 eventhandlers.go:229] add event for scheduled pod default/web-operator-597d4c4d88-9qxkz 
  I0330 01:03:52.403738     820 scheduler.go:731] pod default/web-operator-597d4c4d88-rnh7g is bound successfully on node "node2", 3 nodes evaluated, 3 nodes were found feasible.
  I0330 01:03:52.405018     820 scheduler.go:731] pod default/web-operator-597d4c4d88-9qxkz is bound successfully on node "node2", 3 nodes evaluated, 3 nodes were found feasible.
  I0330 01:03:52.405999     820 scheduler.go:731] pod default/web-operator-597d4c4d88-5ndwn is bound successfully on node "node2", 3 nodes evaluated, 3 nodes were found feasible.
  I0330 01:03:52.419163     820 eventhandlers.go:205] delete event for unscheduled pod default/web-operator-597d4c4d88-rnh7g
  I0330 01:03:52.419273     820 eventhandlers.go:205] delete event for unscheduled pod default/web-operator-597d4c4d88-5ndwn
  I0330 01:03:52.419312     820 eventhandlers.go:229] add event for scheduled pod default/web-operator-597d4c4d88-rnh7g 
  I0330 01:03:52.419343     820 eventhandlers.go:229] add event for scheduled pod default/web-operator-597d4c4d88-5ndwn 
  ```

- 牛刀小试 --修改修改kubernetes kube-schduler源码为了更好的提现修改了代码这里仅仅修改一行注释

- 

- ```go
if klog.V(2) {
  				klog.Infof("pod %v/%v is bound successfully on node %q, %d nodes evaluated, %d nodes were found feasible.", assumedPod.Namespace, assumedPod.Name, scheduleResult.SuggestedHost, scheduleResult.EvaluatedNodes, scheduleResult.FeasibleNodes)
  				klog.Infof("pod %v/%v 成功调度到node工作节点 %q, %d nodes 进行评估, %d nodes 是可用的.", assumedPod.Namespace, assumedPod.Name, scheduleResult.SuggestedHost, scheduleResult.EvaluatedNodes, scheduleResult.FeasibleNodes)
  			}
  ```
```

- k8s源码并重新进行编译 

  ```shell
  # 执行  make 是清除_output/bin 目录下已经编译的二进制文件
  [root@master kubernetes]# make  clean 
  +++ [0330 21:25:29] Verifying Prerequisites....
  +++ [0330 21:25:32] Removing _output directory
  Removing test/e2e/generated/bindata.go ..
  
  #重新编译 只编译kube-schduler组件
  
  
```

- 重新执行上面的  启动监听端口 、 删除pod 测试、查看日志

  ```shell
  
  ```

  

- 恢复集群

  ```shell
  [root@master manifests]# mv   ../kube-scheduler.yaml   .
  ```

  ```go
  # kube-scheduler.go 
  # 实例化 command
  command := app.NewSchedulerCommand()
  # 运行kube-scheduler 逻辑
  Run: func(cmd *cobra.Command, args []string) {
  			if err := runCommand(cmd, args, opts, registryOptions...); err != nil {
  				fmt.Fprintf(os.Stderr, "%v\n", err)
  				os.Exit(1)
  			}
  		},
  #启动 schduler
  
  
  
  
  ```
  
  

###### 3.2  kube-apiservere  调式

- 组件介绍kube-apiserver 为负责提供对外api组件
- 代码执行流程
- 以下过程都是1. goland  remote配置 2. 开启监听服务 3. goland开启监听 4. 测试

```

```



###### 3.3  kube-controllermanger  调式

kube-controllermanager 调试 

- 3.3.1编译kube-controller-manager 源码

```shell
[root@master kubernetes-v1.18.5]# GO111MODULE=off KUBE_GIT_TREE_STATE=clean KUBE_GIT_VERSION=v1.18.5  KUBE_BUILD_PLATFORMS=linux/amd64 make WHAT=cmd/kube-controller-manager  GOGCFLAGS="all=-N -l"+++    
warning: ignoring symlink /home/gowork/src/k8s.io/kubernetes-v1.18.5/_output/local/go/src/k8s.io/kubernetes
go: warning: "k8s.io/kubernetes/vendor/github.com/go-bindata/go-bindata/..." matched no packages
+++ [0411 10:52:41] Building go targets for linux/amd64:
    cmd/kube-controller-manager

```

- 3.3.2 移除正在服务的  kube-controller-manager  组件

  ```shell
  [root@master kubernetes]# mv  manifests/kube-controller-manager.yaml   .
  ```

  

- 3.3.3配置  goland

  ```
  ...
  ```



-  3.3.4 远程开启服务

  ```shell
  [root@master kubernetes]# dlv exec _output/bin/kube-controller-manager --headless -l 192.168.108.130:2345 --api-version=2 -- \
  --allocate-node-cidrs=true \
  --authentication-kubeconfig=/etc/kubernetes/controller-manager.conf \
  --authorization-kubeconfig=/etc/kubernetes/controller-manager.conf \
  --bind-address=127.0.0.1 \
  --client-ca-file=/etc/kubernetes/pki/ca.crt \
  --cluster-cidr=10.244.0.0/16 \
  --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt \
  --cluster-signing-key-file=/etc/kubernetes/pki/ca.key \
  --controllers=*,bootstrapsigner,tokencleaner \
  --kubeconfig=/etc/kubernetes/controller-manager.conf \
  --leader-elect=true \
  --node-cidr-mask-size=24 \
  --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt \
  --root-ca-file=/etc/kubernetes/pki/ca.crt \
  --service-account-private-key-file=/etc/kubernetes/pki/sa.key \
  --service-cluster-ip-range=10.96.0.0/12 \
  --use-service-account-credentials=true
  ```

  ![1618110654475](D:\workspace\GoWorks\src\github.com\k8s-dev-record\chapter one\png\1618110654475.png)

- 3.3.5 测试本地服务的正常工作

  ```shell
  [root@master install]# kubectl  delete po -l name=web-operator
  pod "web-operator-597d4c4d88-m4h5q" deleted
  pod "web-operator-597d4c4d88-t7t8l" deleted
  pod "web-operator-597d4c4d88-zt5r4" deleted
  [root@master install]# kubectl get  po --show-labels          
  NAME                            READY   STATUS              RESTARTS   AGE     LABELS
  test-pod                        0/1     Completed           0          4m57s   <none>
  web-operator-597d4c4d88-857qh   1/1     Running             0          72s     name=web-operator,pod-template-hash=597d4c4d88
  web-operator-597d4c4d88-mh7fb   1/1     Running             0          72s     name=web-operator,pod-template-hash=597d4c4d88
  web-operator-597d4c4d88-vw969   0/1     ContainerCreating   0          72s     name=web-operator,pod-template-hash=597d4c4d88
  ```

  

###### 3.4 kube-proxy  调式

kube-proxy  组件介绍：

###### 3.5 kubelet 调试

```
kubelet 组件最为复杂,
- kubelet 组件功能介绍
- 代码执行流程
配置golang
```

