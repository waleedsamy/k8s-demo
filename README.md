# k8s-demo
Hit kubernetes, to bootstrap kubernetes you can check [k8s on ubuntu with Vagrant and Ansible](https://github.com/waleedsamy/k8s-ubuntu) or use [kubeadm](https://kubernetes.io/docs/getting-started-guides/kubeadm/)

### Run, show, scale and expose to public
```bash
 kubectl create namespace infra
 kubectl create namespace green-dev
 kubectl label node 172.9.8.51 role=ingress
 kubectl create -f ./deploy/ingress.yaml
 # for more details about exposed metrics [check](https://github.com/kubernetes/kube-state-metrics#metrics)
 kubectl create -f ./deploy/kube-state-metrics.yaml
 # prometheus configuration is based on official [sample](https://github.com/prometheus/prometheus/blob/master/documentation/examples/prometheus-kubernetes.yml)
 kubectl create  -f ./deploy/prometheus.yaml
 kubectl create  -f ./deploy/prometheus-node-exporter.yaml
 kubectl create  -f ./deploy/grafana.yaml
 # elasticsearch
 kubectl create  -f ./deploy/es.yaml
 # Need to configure env SLACK_WEBHOOK_URL and SLACK_CHANNEL otherwise use waleedsamy/k8s-fluentd-nodejs-winston:v1.0 instead
 kubectl create -f ./deploy/fluentd-daemonSet.yaml
 kubectl create -f ./deploy/hello.yaml
 kubectl create  -f ./deploy/kibana.yaml
```

### View your cluster
```bash
# run weavescope on k8s, check Utils for port forwarding to be able to access from your browser
kubectl create -f 'https://cloud.weave.works/launch/k8s/weavescope.yaml' --validate=false
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
kubectl run -i --tty curl --image=radial/busyboxplus:curl
# port forward to pod, you now can access it as if it is running in your machine, usually use it if I need to access service in my browser
kubectl port-forward {pod} 4040
# select pod name
kubectl get pod --selector=weavescope-component=weavescope-app -o jsonpath='{.items..metadata.name}'
# forward port to prometheus pod
kubectl --namespace=infra get pods -l app=prometheus -o name | \
	sed 's/^.*\///' | \
	xargs -I{} kubectl --namespace=infra port-forward {} 9090:9090
# access kubernetes from a pod using default serviceaccount
curl --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt  -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" https://kubernetes.default.svc.cluster.local
```

```bash
# when creating container with --net-alias, it can be reached with this name from other containers in the same network
docker network create green
docker run -d --net=green --net-alias web nginx:alpine
docker run --rm --net green lherrera/bind-tools host -t A web
docker run --rm --net green lherrera/bind-tools ping web
```


#### Notes:
  * kubernetes service should has `prometheus.io/probe` with value of true to be scraped
  * kubernetes pod scraping is configured by:
    * `prometheus.io/scrape`: Only scrape pods that have a value of `true`
    * `prometheus.io/path`: If the metrics path is not `/metrics` override this.
    * `prometheus.io/port`: Scrape the pod on the indicated port instead of the default of `9102`.

* resources:
 * http://blog.oestrich.org/2016/01/nodeport-kubernetes-load-balancer/
