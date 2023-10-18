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

**1-Install NGINX Ingress Controller (using the official manifests and apply above config )**
```
kubectl apply -f ingress-deploy.yaml
```

**2-Check that the Ingress controller pods have started**
```
➤ kubectl get pods -n ingress-nginx
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-txj28        0/1     Completed   0          26h
ingress-nginx-admission-patch-w6l44         0/1     Completed   2          26h
ingress-nginx-controller-6f4df7b5d6-cch5n   1/1     Running     0          26h
```
**3-Check that you can see the LoadBalancer service**
```
~➤ kubectl get svc -n ingress-nginx
NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.109.151.57    192.168.86.184   80:32090/TCP,443:32307/TCP   26h
ingress-nginx-controller-admission   ClusterIP      10.108.157.170   <none>           443/TCP                      26h
```
**4-From version v1.0.0 of the Ingress-NGINX Controller, a ingressclass object is required.**

In the default installation, an ingressclass object named nginx has already been created.
If this is only instance of the Ingresss-NGINX controller, you should add the annotation ingressclass.kubernetes.io/is-default-class in your ingress class:
```
kubectl -n ingress-nginx annotate ingressclasses nginx ingressclass.kubernetes.io/is-default-class="true"
```
OR

```
kubectl edit ingressclasses.networking.k8s.io nginx

 annotations:
    ingressclass.kubernetes.io/is-default-class: "true"
```
**Show Log**
```
kubectl logs -f -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx
```
**To work routing path properly **
use bellow config in ingress in order to routing path work properly
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
    nginx.ingress.kubernetes.io/use-regex: "true"
spec:
  rules:
    - host: front-end.info
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: front-end
                port:
                  number: 8081
    - host: broker-service.info
      http:
        paths:
          - path: /(.*)
            pathType: Prefix
            backend:
              service:
                name: broker-service
                port:
                  number: 8080

```
The annotation `nginx.ingress.kubernetes.io/rewrite-target: /$1` is used in Kubernetes Ingress configurations with the NGINX Ingress Controller. It indicates that when a request matches a specific Ingress rule, the path of the request should be rewritten. The `$1` represents a placeholder for the captured group from the regular expression used in the Ingress rule.

In this case, if a request comes in with a path such as `/foo/bar`, the NGINX Ingress Controller will rewrite the path to `/bar` and forward the request to the backend service named `my-service`.

```
kubectl apply -f ingress.yaml
kubectl apply -f mailhog-ingress.yaml
```

