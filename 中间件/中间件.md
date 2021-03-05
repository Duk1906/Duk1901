##### Consul
1. 一个基于raft协议实现的配置管理组件，支持多数据中心，可以轻松支持上千台服务器的配置管理工作
2. 主要包含2种角色，client和server，其中server又分为follower和leader，所有读写请求都会转发给leader，从而确保数据读写的一致性
3. 选主依据：时代、事务id、hostip（需要获取超半数投票才可成功选举出leader）
	
##### Zookeeper
1. 基于paxos改进的zab协议实现
2. 也是1 leader + 多个follower组成，同样，所有读写请求都会转发给leader，从而确保数据读写的一致性
3. 选主依据：时代、事务id、server_id（需要获取超半数投票才可成功选举出leader）

##### Nginx
1. 一个基于多进程+Epoll机制实现的高性能负载均衡器，可以支持四七层代理
2. 多进程（充分利用多核优势）
a. 主进程：负责初始化套接字以及管理子worker进程的生命流程
b. worker进程：负责处理具体的请求 
3. Epoll + 边缘触发 机制（解决单线程处理io操作的性能问题）
4. worker进程无任务休眠，有任务会被唤醒；nginx为每个worker进程设置一个全局锁（accept_mutex），当有io事件就绪时，仅获取到锁的worker能够进入epoll队列中处理相应的io任务，从而避免惊群事件发生

##### Keepalived
1. 基于vrrp协议，正常情况下包括一个主进程+2子进程（vrrp子进程负责集群内vrrp通信，check子进程负责衍生更多的孙子进程，检查vrrp实例的健康状态）
2. 选主（vip是否绑在某主机上-----配置的state/master或slave----priority----hostip）
	
##### Supervisor
1. 一个基于python开发的任务管理工具，配置简单，使用方便
2. 本身的可用性可以由crontab保障
3. systemd，系统1号进程，配置相对复杂，但Linux系统会保障systemd的可用性
4. 一句话概括：可以用systemd管理supervisor
