SETUP:
sudo hostnamectl set-hostname "k8s-master"
exec bash
sudo apt-get install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
sudo apt-get install apt-transport-https curl -y
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
swapoff -a    # will turn off the swap
sudo apt-get install kubeadm -y

sudo kubeadm init --apiserver-advertise-address=192.168.1.25 --pod-network-cidr=10.244.0.0/16 --apiserver-cert-extra-sans=kubectl.tk,192.168.1.25
flannel:
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
kubeadm join 192.168.1.25:6443 --token yq8dl7.pg9zkgrcur1np4uy --discovery-token-ca-cert-hash sha256:71e48983d3fc2f81312924820592449f8defc0fc09bb00d9e158061cbaec9ca2
sudo apt-get install nfs-common

kubeadm reset
systemctl daemon-reload
systemctl restart kubelet
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
sudo apt-get purge kubeadm kubectl kubelet kubernetes-cni kube*   
sudo apt-get autoremove  
sudo rm -rf ~/.kube
kubectl drain <node name> --delete-local-data --force --ignore-daemonsets
kubectl delete node <node name>

RBAC:
openssl genrsa -out mra.key 2048
openssl req -new -key mra.key -out mra.csr -subj "/CN=mra/O=development"
openssl x509 -req -in mra.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out mra.crt -days 365
kubectl --kubeconfig mra.config config set-cluster kubernetes --server https://192.168.1.25:6443 --certificate-authority ca.crt
kubectl --kubeconfig mra.config config set-credentials mra --client-certificate /home/u-user/kube/mra.crt --client-key /home/u-user/kube/mra.key
kubectl --kubeconfig mra.config config set-context mra-kubernetes --cluster kubernetes --namespace development --user mra
kubectl create role mra.development --verb list,get,create,delete --resource '*' --namespace development
kubectl create rolebinding mra-development-rolebinding --role mra.development --user mra -n development
cat /home/u-user/kube/mra.crt | base64 -w0
MONITORING:
 helm  inspect values stable/prometheus > /home/u-user/prometheus.values
 helm install prometheus stable/prometheus --values /home/u-user/prometheus.values -n prometheus
 helm install heapster stable/heapster --values heapster.values 
CI/CD:
docker build -t kube/jenkins-image:1.0 .
docker tag kube/jenkins-image:1.01 192.168.1.25:5000/jenkins-image:1.01


Short name	Full name
csr      	certificatesigningrequests
cs	componentstatuses
cm	configmaps
ds	daemonsets
deploy	deployments
ep	endpoints
ev	events
hpa horizontalpodautoscalers
ing	ingresses
limits	limitranges
ns	namespaces
no	nodes
pvc	persistentvolumeclaims
pv	persistentvolumes
po	pods
pdb	poddisruptionbudgets
psp	podsecuritypolicies
rs	replicasets
rc	replicationcontrollers
quota	resourcequotas
sa	serviceaccounts
svc	services
