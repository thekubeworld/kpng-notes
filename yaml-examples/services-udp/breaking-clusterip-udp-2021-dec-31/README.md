# Creating the service

Name: svc-udp
Namespace: conntrack-3252  
Type: ClusterIP  
Port: 80  
Protocol: UDP  

Which E2E is this failing?: 
"should be able to preserve UDP traffic when server pod cycles for a ClusterIP service"


```
$ kubectl apply -f svc-udp-crash.yaml
```

# Listening the port 80
```
tcpdump 'port 80'
```

# Listining the service
```
$ kubectl get svc -n conntrack-3252
NAME      TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
svc-udp   ClusterIP   10.96.88.17   <none>        80/UDP    3m45s
```

# Service YAML
```
$ kubectl get svc -n conntrack-3252 -o yaml
apiVersion: v1
items:
- apiVersion: v1
  kind: Service
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"creationTimestamp":"2021-12-31T14:13:03Z","labels":{"testid":"svc-udp-d11ffb2a-94b2-4a8d-816b-3e84f35794b1"},"name":"svc-udp","namespace":"conntrack-3252","resourceVersion":"788","uid":"811c36e8-5f0d-47a7-a7af-06a6bb009721"},"spec":{"clusterIP":"10.96.88.17","clusterIPs":["10.96.88.17"],"internalTrafficPolicy":"Cluster","ipFamilies":["IPv4"],"ipFamilyPolicy":"SingleStack","ports":[{"name":"udp","port":80,"protocol":"UDP","targetPort":80}],"selector":{"testid":"svc-udp-d11ffb2a-94b2-4a8d-816b-3e84f35794b1"},"sessionAffinity":"None","type":"ClusterIP"},"status":{"loadBalancer":{}}}
    creationTimestamp: "2021-12-31T14:17:16Z"
    labels:
      testid: svc-udp-d11ffb2a-94b2-4a8d-816b-3e84f35794b1
    name: svc-udp
    namespace: conntrack-3252
    resourceVersion: "1350"
    uid: 19bfe273-bc15-4d39-afb4-f2ee2ff7553c
  spec:
    clusterIP: 10.96.88.17
    clusterIPs:
    - 10.96.88.17
    internalTrafficPolicy: Cluster
    ipFamilies:
    - IPv4
    ipFamilyPolicy: SingleStack
    ports:
    - name: udp
      port: 80
      protocol: UDP
      targetPort: 80
    selector:
      testid: svc-udp-d11ffb2a-94b2-4a8d-816b-3e84f35794b1
    sessionAffinity: None
    type: ClusterIP
  status:
    loadBalancer: {}
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
```

# kpng ipvs logs
```
# k logs -f -n kube-system kpng-qmwxg kpng-ipvs
I1231 14:02:42.440198       1 ipvs.go:110] creating dummy interface kube-ipvs0
I1231 14:02:42.440716       1 ipvs.go:122] setting dummy interface kube-ipvs0 up
W1231 14:02:42.441127       1 ipset.go:165] ipset name truncated; [KUBE-6-LOAD-BALANCER-SOURCE-CIDR] -> [KUBE-6-LOAD-BALANCER-SOURCE-CID]
W1231 14:02:42.441156       1 ipset.go:165] ipset name truncated; [KUBE-6-NODE-PORT-LOCAL-SCTP-HASH] -> [KUBE-6-NODE-PORT-LOCAL-SCTP-HAS]
I1231 14:02:42.465397       1 ipvs.go:307] SetEndpoint("default", "kubernetes", "116724088420fc51", IPs:{V4:"172.18.0.7"})
I1231 14:02:42.465501       1 ipvs.go:307] SetEndpoint("kube-system", "kube-dns", "123dcf68b0568cc", IPs:{V4:"10.244.1.3"})
I1231 14:02:42.465567       1 ipvs.go:307] SetEndpoint("kube-system", "kube-dns", "1304f720c87c4a7b", IPs:{V4:"10.244.1.4"})
I1231 14:03:38.614629       1 ipvs.go:307] SetEndpoint("conntrack-1478", "svc-udp", "b0616e0e8a5411ad", IPs:{V4:"10.244.2.2"} Local:true)
I1231 14:03:51.641981       1 ipvs.go:307] SetEndpoint("conntrack-1478", "svc-udp", "9b0f5a73decdceaf", IPs:{V4:"10.244.2.3"} Local:true)
I1231 14:03:52.558803       1 ipvs.go:326] DeleteEndpoint("conntrack-1478", "svc-udp", "b0616e0e8a5411ad")
I1231 14:04:57.689836       1 ipvs.go:326] DeleteEndpoint("conntrack-1478", "svc-udp", "9b0f5a73decdceaf")
```

# ginkgo command
```

$ kpng/hack/temp/e2e> ./ginkgo --nodes="26" \
       --focus="should be able to preserve UDP traffic when server pod cycles for a ClusterIP service" \
       --skip="Feature|Federation|PerformanceDNS|Disruptive|Serial|LoadBalancer|KubeProxy|GCE|Netpol|NetworkPolicy" \
        ./e2e.test \
        -- \
        --kubeconfig="kubeconfig_tests.conf" \
        --provider="local" \
        --dump-logs-on-failure=false \
        --report-dir="artifacts/reports" \
        --disable-log-dump=true
```


# Alpine machines add tcpdump pkg
```
# apk add tcpdump
# tcpdump 'port 80'
```
