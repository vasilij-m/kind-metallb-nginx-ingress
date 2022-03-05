# KIND cluster with Metallb and NGINX Ingress controller

1. **Create cluster**
 
```bash
kind create cluster --config ./kind-config.yaml
```
2. **Deploy metallb**

Create metallb namespace:

```bash
kubectl apply -f ./metallb-namespace.yaml
```

A range of IP addresses for metallb ConfigMap must be within the docker kind network. We can get docker kind network CIDR with this command:

```bash
docker network inspect -f '{{.IPAM.Config}}' kind
```

Deploy metallb:

```bash
kubectl apply -f ./metallb
```
Wait for metallb pods to have a status of Running:

```bash
kubectl get pods -n metallb-system --watch
```

3. **Deploy NGINX Ingress controller**
   
```bash
kubectl apply -f ./ingress
```

Wait for ingress-nginx-controller-* pods to have a status of Running:

```bash
kubectl get pods -n ingress-nginx --watch
```

Verify that ingress-nginx-controller service get IP address from metallb address pool:

```bash
kubectl -n ingress-nginx get svc
NAME                                 TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.96.251.111   172.19.100.100   80:30717/TCP,443:30962/TCP   35m
ingress-nginx-controller-admission   ClusterIP      10.96.132.149   <none>           443/TCP                      35m
```

4. **Deploy test app**

```bash
kubectl apply -f ./ingress-usage-example.yaml 
```

Verify that the ingress works:

```bash
# should output "foo"
curl 172.19.100.100/foo
# should output "bar"
curl 172.19.100.100/bar
```
