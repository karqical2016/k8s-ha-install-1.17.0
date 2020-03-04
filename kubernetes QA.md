Q:   控制Master节点是否可调度

A:   允许 kubectl taint nodes --all node-role.kubernetes.io/master- 
    

 禁止 kubectl taint nodes centos-master-1 node-role.kubernetes.io/master=true:NoSchedule

Q:   Jan 11 09:42:40 k8s78 kubelet[517]: E0111 09:42:40.935017     517 summary.go:92] Failed to get system container stats for "/system.slice/docker.service": failed to get cgroup stats for "/system.slice/docker.service": failed to get container info for "/system.slice/docker.service": unknown container "/system.slice/docker.service"

A:   在kubelet中追加配置
--runtime-cgroups=/sys/fs/cgroup/systemd/system.slice --kubelet-cgroups=/sys/fs/cgroup/systemd/system.slice

Q:   Failed to list *v1.Node: nodes is forbidden

A:   认证加Node，策略加NodeRestriction
--authorization-mode=Node,RBAC \
--admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota,DefaultTolerationSeconds,NodeRestriction 

Q:   iptables中会自动增加服务的reject问题

A:   关于iptables规则中的KUBE-FIREWALL并不会影响服务，影响的主要原因是如下策略，该规则是动态产生不人为控制，
这也是kubernetes内部的熔断机制（类似于nginx的健康检查,k8s中的service也有），即当的服务中(只限有endpoint的服务)的所有容器都无法请求时，会自动增加reject做内部防护

Q:   Failed to start container manager: inotify_add_watch /sys/fs/cgroup/cpuacct,cpu: no such file or directory

A:   最新版cAdvisor的bug，记录下workaround
mount -o remount,rw '/sys/fs/cgroup'
ln -s /sys/fs/cgroup/cpu,cpuacct /sys/fs/cgroup/cpuacct,cpu

Q:   Failed to start cAdvisor inotify_add_watch /sys/fs/cgroup/cpu,cpuacct/system.slice/run-26637.scope: no space left on device

A:   cat /proc/sys/fs/inotify/max_user_watches # default is 8192
sudo sysctl fs.inotify.max_user_watches=1048576 # increase to 1048576

Q:   unable to get metrics for resource memory: unable to fetch metrics from resource metrics API: the server could not find the requested resource (get pods.metrics.k8s.io)

A:   [ips@ips81 bin]$ curl 10.254.16.15:18082/apis/metrics/v1alpha1/nodes
      curl: (7) Failed connect to 10.254.16.15:18082; Connection refused
     [ips@ips81 bin]$ ./kubectl top node
     Error from server (ServiceUnavailable): the server is currently unable to handle the request (get services http:heapster:)

    1.由于flannel网络不通
    2./kubectl -n kube-system get ep 查看端口是否正确

Q:   Metric-server: x509: subject with cn=front-proxy-client is not in the allowed list: [aggregator]

A:   请求头部标识不正确，在kube-apiserver中增加配置
     --requestheader-allowed-names=aggregator,front-proxy-client 

Q:  failed to register unfinished metric admission_quota_controller: duplicate metrics collector registration attempted

