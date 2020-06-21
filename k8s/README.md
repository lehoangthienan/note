### gen certs
sudo mkdir certs
sudo chmod 777 -R certs
openssl req -nodes -newkey rsa:2048 -keyout certs/dashboard.key -out certs/dashboard.csr -subj "/C=/ST=/L=/O=/OU=/CN=kubernetes-dashboard"
openssl x509 -req -sha256 -days 365 -in certs/dashboard.csr -signkey certs/dashboard.key -out certs/dashboard.crt
sudo chmod -R 777 certs

### create secret
kubectl create secret generic kubernetes-dashboard-certs --from-file=certs -n kubernetes-dashboard

### metric k8s
git clone https://github.com/kubernetes-sigs/metrics-server && cd metrics-server
change config to add:
args:
           - --kubelet-insecure-tls
           - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname

run pod and healthcheck: kubectl top node

### dashboard k8s
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-rc2/aio/deploy/recommended.yaml

### get token dashboard k8s
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')

### url dashboard k8s
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

### start proxy
kubectl proxy || kubectl proxy --port=8080

### access to cluster ip with proxy
http://localhost:8080/api/v1/proxy/namespaces/<NAMESPACE>/services/<SERVICE-NAME>:<PORT-NAME>/

### NGINX Ingress
git clone git@github.com:nginxinc/kubernetes-ingress.git
cd kubernetes-ingress/deployments

kubectl apply -f common/ns-and-sa.yaml
kubectl apply -f common/default-server-secret.yaml
kubectl apply -f common/nginx-config.yaml
kubectl apply -f rbac/rbac.yaml
kubectl apply -f daemon-set/nginx-ingress.yaml

Check: kubectl get ds -n nginx-ingress
kubectl get po -n nginx-ingress

### Scale replicas
kubectl scale deploy/skinapp --replicas=10

## auto scale pod
kubectl autoscale deploy/skinapp --min=2 --max=5 --cpu-percent=50

## auto scale node
https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-autoscaler

### k9s
- macos:  brew install derailed/k9s/k9s
- linux: sudo apt update && sudo apt install snapd && sudo snap install k9s

### SSL/TSL AUTO
https://github.com/jetstack/cert-manager