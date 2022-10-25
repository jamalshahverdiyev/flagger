#### Commands

#### Install flagger:
```bash
$ helm repo add flagger https://flagger.app
$ helm upgrade -i flagger flagger/flagger \
    --namespace=istio-system \
    --set crd.create=false \
    --set meshProvider=istio \
    --set metricsServer=http://prometheus:9090
$ helm upgrade -i flagger-loadtester flagger/loadtester --namespace=istio-system --set cmd.timeout=1h
```

#### Execute deployment commands and create flagger CRD:
```bash
$ kubectl create ns company-mss && kubectl label namespace company-mss istio-injection=enabled
$ docker login registry.gitlab.com
$ kubectl create secret generic gitlabregistry --from-file=.dockerconfigjson=/root/.docker/config.json --type=kubernetes.io/dockerconfigjson -n company-mss
secret/gitlabregistry created
$ kubectl apply -f db/pv.yaml && kubectl apply -f db/pvc.yaml
$ kubectl apply -f db/svc.yaml -f db/deployment.yaml
$ kubectl apply -f app/hpa.yaml -f app/deployment.yaml
$ kubectl apply -f app/flagger.yaml
```

#### Apply new deployments and look at the tmux sessions:
```bash
$ kubectl apply -f app/hpa.yaml -f app/deployment-v2.yaml
$ kubectl apply -f app/hpa.yaml -f app/deployment-v3.yaml
$ kubectl apply -f app/hpa.yaml -f app/deployment-v1.yaml
```

#### Tmux commands:
```bash
$ watch -n1 'kubectl describe canaries.flagger.app -n company-mss ms-auth | tail -n2'
$ watch -n1 'kubectl get canaries.flagger.app -n company-mss'
$ watch -n1 'kubectl get svc,pods,deployments -n company-mss --no-headers'
$ kubectl logs -f $(kubectl get pods -n istio-system| grep flagger-loadtester | awk '{ print $1 }') -n istio-system
```

#### Inside of some container execute curl:
```bash
$ kubectl exec -it $(kubectl get pods | grep api-gateway) -c api-gateway -- sh
$ apk update && apk add curl
$ for i in `seq 10000`; do curl -XGET http://ms-auth.company-mss/version/endpoint; sleep 1; done
$ kubectl logs -f deployments/ms-auth-primary -n company-mss -c ms-auth  
```
