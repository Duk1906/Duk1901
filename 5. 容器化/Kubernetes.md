```shell
一. 网络实现（2020-10-12 ~ 10.18）
     说明：node可部署多个pod，pod内可包含多个container
     1.  同一pod内container同享同一网络命名空间，所以同一pod内container间可通过localhost通信
     2.  同一node不通pod通信：pod1/vethx------vethx[Docker0网桥]vethp------vethp/pod2
     3.  不同node上pod通信：
          [host1]pod1/vethx---[host1]vethx[Docker0网桥]---[host1]eth0<--->[host2]eth0---[host2]vethL[Docker0网桥]---[host2]vethL/pod2
     4. CNI网络模型：Container Network Interface，对容器网络进行操作和配置的规范
     5. NetworkPolicy：对pod间的网络通信进行限制及准入控制（查询级别包含 pod、namespace）
     6. Kubernetes相关开源的网络组件（下面几类主要功能是，让所有pod处于一个可直接连通的网络空间内）
         - Flannel：路由信息存在公共的etcd中，管理简单且不易冲突；但数据传输从flannel0网络接口转到用户态flanneld程序，接收相反，会带来一定的时延损耗
                 hostl [ docker0/flannel0---flanneld[etcd]---eth0 ] <-----> hostx [ eth0---flanneld[etcd]--docker0/flannel0 ]
         - Calico：在每个node上部署 Calico Agent、BGP Client
               * Felix/Calico Agent：负责为 container 设置网络资源（ip地址、路由规则、iptables规则等）
               * BGP Client：负责把 Felix 设置网络信息通过 BGP 协议广播到 Calico网络
               * Router Reflector：完成大规模集群的分级路由分发
               * Etcd：Calico使用的后端存储

二. Service（2020-10-19 ~ 10.25）
	1. 功能：可以为一组具有相同功能的pod提供一个统一的入口地址
	2. 创建方式：
	kubectl expose rc zk
	Kubectl create -f zk_srv.yaml
	3. 负载均衡方式：
	RoundRobin：轮询模式，默认
	SessionAffinity：基于客户端ip进行会话保持的模式，设置service.spec.sessionAffinity=ClientIP即可
	4. 外部服务service
	>  将外部服务设置为service的后端代理
	A. 创建一个无label selector的service
	B. 创建同名endpoint
	kind: Endpoints
	apiVersion: v1
	metadata:
	    name: $same_name
	subsets:
	- addresses
	  - ip: $proxy_ip
	  port:
	  - port: $proxy_port
	5. Headless Service
	> clusterIP为None，仅设置label Selector将后端pod列表返回给调用的客户端
	6. 从外部访问pod
	> 将pod的端口映射到物理机上(在pod的container模板中如下定义)
	- containerPost: $ct_post
	  hostPort: $h_post
	7. 从外部访问Service
	> 将Service的端口映射到物理机上
	Kind: Service
	spec:
	   type: NodePort
	   ports:
	   - post: $service_port
	     targetPort: $pod_port
	     nodePort: $host_port

三. DNS（2020-10-19 ~ 10.25）
	
   A.  Service级别的DNS配置
	1. SkyDNS架构（kubernetes 1.2版本）
	• etcd：存储 DNS 记录
	• kube2sky：通过 k8s master 获取/监控集群内 Service 资源变化，将 Service 的名称及 ip 信息生成 DNS 记录，保存在 etcd 中
	• skydns：从 etcd 中读取 DNS 记录，为客户端容器应用提供 DNS 查询服务
	• heathz：对 skydns 服务进行健康检查
	
	2. KubeDNS架构（kubernetes 1.4版本）
	• kubedns：通过 k8s master 获取/监控集群内 Service 资源变化，将 Service 的名称及 ip 信息生成 DNS 记录，保存在内存中
	• dnsmasq：从 kubedns 中获取 DNS 记录，提供 DNS 缓存，为客户端容器应用提供 DNS 查询服务
	• sidecar：对 kubedns 和 dnsmasq 进行健康检查。 
	
	3. CodeDNS架构（kubernetes 1.11版本）
	• coredns：集 KubeDNS 3个容器组件功能于一身

   B. pod级别的DNS配置说明
	1. Pod YAML 中的 spec.dnsPolicy:
	• Default：继承 pod 所在宿主机的 dns 设置
	• ClusterFirst：优先使用 kubernetes 环境的 DNS 服务，无法解析的域名转发到宿主机继承的 DNS 服务器
	• ClusterFirstWithHostNet：与 ClusterFirst 相同，对于以 hostNetwork 模式运行的 Pod，应明确指定该策略
	• Node：通过 spec.dnsConfig 自定义的 DNS 配置，不再使用 kubernetes 环境的 DNS服务
	spec.dnsConfig:
	* nameservers: 一组dns服务器的列表，最多设置3个
	* searches：一组用于域名搜索的 DNS 域名后缀，最多设置6个
	* option：配置可选的DNS参数，例如 ndots、timeout等，以name或name/value表示
	
四. Ingress（2020-10-19 ~ 10.25）
	A. Ingress简介
	• Service仅能提供ip:port的负载均衡方式，即，工作在TCP/IP层
	• Ingress可以基于不同URL将请求转发至后端不同的Service，实现HTTP层的业务路由机制
	B. 使用方式
	1. 创建 Ingress Controller（以pod方式运行，监控API server的/ingress接口的后端backend services，本质是一个nignx http层的负载均衡器）
	可以将 Ingress Controller 以 daemonset 方式创建，即在每个node上启动一个Nginx服务，设置hostPost，将容器应用端口映射在物理机上，客户端应用就可以访问了
	2. 为Ingress Controller设置一个默认的backend Service，充当404 not found的功效
	3. 定义Ingress 策略
	C. 策略配置
	• 转发到单个后端服务上
	kind: Ingress
	spec:
	   backend:
	      serviceName: $bk_srv
	      servicePort: $post
	• 相同域名，不同url转发到不同服务上
	kind: Ingress
	spec:
	   rules:
	    - host: $domain.com
	      http:
	         paths:
	         - path: $url1 
	            backend:
	                 serviceName: $bk_srv1
	                 servicePort: $post1
	         - path: $url2
	            backend:
	                 serviceName: $bk_srv2
	                 servicePort: $post2
	• 不同域名转发到不同服务上
	kind: Ingress
	spec:
	   rules:
	    - host: $domain.com1
	       http:
	          paths:
	             backend:
	                 serviceName: $bk_srv1
	                 servicePort: $post1
	    - host: $domain.com2
	       http:
	          paths:
	             backend:
	                 serviceName: $bk_srv2
	                 servicePort: $post2
	• 不使用域名的转发规则
	> 访问方式：curl -k https://ingress-controller-ip/url . 默认禁止非安全的http，强制启动https
	kind: Ingress
	spec:
	   rules:
	       - http:
	          paths:
	          - path: $url1 
	             backend:
	                 serviceName: $bk_srv1
	                 servicePort: $post1
	> 关闭强制启动https设置
	kind: Ingress
	metadata：
	     annotations:
	         ingress.kubernetes.io/ssl-redirect: "false"
	D. TLS安全设置(步骤)
	1. 创建自签名的密钥和ssl证书文件
	2. 将证书保存在Kubernetes中的一个Secret资源对象上
	3. 将该Secret对象设置到ingress中

```

