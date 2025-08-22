# 1.1 Management cluster z wystawionym 80/443 na localhost
k3d cluster create mgmt \
--servers 1 --agents 2 \
-p "80:80@loadbalancer" -p "443:443@loadbalancer" \
--k3s-arg "--disable=servicelb,metrics-server@server:0" \
--wait

# 1.2 Trzy klastry robocze (downstream)
k3d cluster create c1 --servers 1 --agents 1 --wait
k3d cluster create c2 --servers 1 --agents 1 --wait
k3d cluster create c3 --servers 1 --agents 1 --wait

# 1.3 Sprawdź konteksty kubectl
kubectl config get-contexts


kubectl config use-context k3d-mgmt


helm repo add jetstack https://charts.jetstack.io
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
helm repo update


helm install cert-manager jetstack/cert-manager \
--namespace cert-manager --create-namespace \
--set crds.enabled=true


kubectl create namespace cattle-system

helm install rancher rancher-latest/rancher \
--namespace cattle-system \
--set hostname=rancher.localhost.sslip.io \
--set replicas=1 \
--set bootstrapPassword=admin


kubectl -n cattle-system rollout status deploy/rancher


kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.yaml
