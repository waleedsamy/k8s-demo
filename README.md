# k8s-demo
Hit kubernetes, to bootstrap kubernetes you can check [k8s on ubuntu with Vagrant and Ansible](https://github.com/waleedsamy/k8s-ubuntu)

### Run, show, scale and expose to public
```
kubectl run api-deployment --image=waleedsamy/hello-world-expressjs-docker --replicas=2 --port=8080
kubectl get pods --show-labels
kubectl scale --replicas=3 deployment/api-deployment
kubectl expose deployment api-deployment --port=8080 --type=LoadBalancer --external-ip=172.17.4.101
```

### View your cluster
```bash
# run weavescope on k8s, check Utils for port forwarding to be able to access from your browser
kubectl create -f 'https://cloud.weave.works/launch/k8s/weavescope.yaml' --validate=false
```


### Ingress
```bash
# pick one of the next reverse proxies, I prefer traefik!
kubectl create -f https://raw.githubusercontent.com/kubernetes/contrib/master/ingress/controllers/nginx-alpha/rc.yaml
kubectl create -f https://raw.githubusercontent.com/containous/traefik/master/examples/k8s.rc.yaml

# allow rule HOST:80 -> ingress:80
sudo iptables -t nat -A  DOCKER -p tcp --dport 80 -j DNAT --to-destination 10.2.24.18:80

```

#### Utils
```bash
# get inside a container
kubectl exec -it {container} -- bash
# curl without need to update your /etc/hosts
curl -H 'Host: foo.example.com' --resolve foo.example.com:80:127.0.0.1 http://foo.example.com/
# run busybox in your cluster
kubectl run -i --tty busybox --image=busybox --restart=Never -- sh; kubectl delete po busybox
$ nslookup kubernetes
# run bind-tools in your cluster, usually i use it to check if DNS work correclt
kubectl run -i --tty bind-tools --image=lherrera/bind-tools --restart=Never -- sh; kubectl delete po bind-tools
$ host -t A kubernetes
# run curl
kubectl run -i --tty curl --image=radial/busyboxplus:curl; kubectl delete deployments curl
# port forward to pod, you now can access it as if it is running in your machine, usually use it if I need to access service in my browser
kubectl port-forward {pod} 4040
# select pod name
kubectl get pod --selector=weavescope-component=weavescope-app -o jsonpath='{.items..metadata.name}'
```

```bash
# when creating container with --net-alias, it can be reached with this name from other containers in the same network
docker network create green
docker run -d --net=green --net-alias web nginx:alpine
docker run --rm --net green lherrera/bind-tools host -t A web
docker run --rm --net green lherrera/bind-tools ping web
```

* resources:
 * http://blog.oestrich.org/2016/01/nodeport-kubernetes-load-balancer/
