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

### NGINX Ingress
https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/cloud/deploy.yaml

git clone https://github.com/nginxinc/kubernetes-ingress/ && cd kubernetes-ingress/deployments
kubectl apply -f common/ns-and-sa.yaml
kubectl apply -f common/default-server-secret.yaml
kubectl apply -f common/nginx-config.yaml
kubectl apply -f rbac/rbac.yaml
kubectl apply -f daemon-set/nginx-ingress.yaml

Check:
kubectl get ds -n nginx-ingress
kubectl get po -n nginx-ingress

### SETUP CLUSTER

# master
kubeadm init --apiserver-advertise-address=172.16.10.100 --pod-network-cidr=192.168.0.0/16
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl apply -f https://docs.projectcalico.org/v3.10/manifests/calico.yaml
kubeadm token create --print-join-command

# worker
kubeadm join ...

# configs
export KUBECONFIG=/Users/anle/.kube/config-mycluster

export KUBECONFIG=~/.kube/config:~/.kube/config-mycluster
kubectl config view --flatten > ~/.kube/config_temp
mv ~/.kube/config_temp ~/.kube/config

kubectl config use-context kubernetes-admin@kubernetes

### cert-manager
