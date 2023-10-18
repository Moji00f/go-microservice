#### Config Ingress on Bare-metal
[Bare-meal](https://github.com/kubernetes/ingress-nginx/blob/main/docs/deploy/baremetal.md#external-ips)

To config ingress withou LoadBalancer like MetalLB we need to set value of two field :
- `externalTrafficPolicy`
- `externalIPs`

when value of `externalTrafficPolicy` field is `Local` !!! be careful This setting effectively **drops packets** sent to Kubernetes nodes which are not running any instance of the NGINX Ingress controller. 

set value of `externalTrafficPolicy` field to `Cluster` in orther to send request fron other server or client outside of cluster

whec run bellow command two field are empty and ingress not work:
- `CLASS`
- `ADDRESS`

```
➤ kubectl get ingress
NAME              CLASS   HOSTS                                ADDRESS          PORTS   AGE
mailhog-ingress   nginx   blog.local                           192.168.86.184   80      19h
my-ingress        nginx   front-end.info,broker-service.info   192.168.86.184   80      19h
```
ingress not expost port to show in output of `netstat`

set value of `externalIPs` field that `ADDRESS` field have IP

!!! example In a Kubernetes cluster composed of 3 nodes (the external IP is added as an example, in most bare-metal environments this value is <None>)

```
➤ kubectl get node
NAME        STATUS   ROLES           AGE    VERSION
master-01   Ready    control-plane   318d   v1.25.4
worker-01   Ready    <none>          317d   v1.25.4
worker-02   Ready    <none>          317d   v1.25.4

spec:
  externalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  externalIPs:
  - 192.168.86.184

type: LoadBalancer

```

```
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.8.2
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  externalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  externalIPs:
  - 192.168.86.184
  ports:
  - appProtocol: http
    name: http
    port: 80
    protocol: TCP
    targetPort: http
  - appProtocol: https
    name: https
    port: 443
    protocol: TCP
    targetPort: https
  selector:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
  type: LoadBalancer
```

seperate mailhog-ingress from ingress.yaml to show mailhog properly 
