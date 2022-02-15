# Short-Scripts
```bash
function install_docker() {
    sudo apt-get -y update
    sudo apt-get -y install docker.io apt-transport-https
    sudo systemctl enable docker.service    
}


function install_k8s() {
   sudo su - -c  "curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -"
   sudo su - -c "echo deb http://apt.kubernetes.io/ kubernetes-xenial main | tee /etc/apt/sources.list.d/kubernetes.list"
   sudo su - -c "apt-get update"
   sudo kubeadm reset -f
   sudo  apt install -y kubelet=1.20.0-00 kubeadm=1.20.0-00 kubectl=1.20.0-00
   sudo kubeadm init
   sudo rm -rf $HOME/.kube
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   sleep 1m
}

function install_helm() {
    sudo rm -rf /usr/local/bin/helm
    ps -ef | grep -v grep | grep -iw helm | awk '{print $2}' | xargs kill -9    
    curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh
    chmod 700 get_helm.sh
    sudo ./get_helm.sh -v v2.17.0

    kubectl -n kube-system create serviceaccount tiller
    kubectl create clusterrolebinding tiller --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
    helm init --service-account tiller
    kubectl taint nodes $(hostname) node-role.kubernetes.io/master-
    kubectl -n kube-system  rollout status deploy/tiller-deploy

    helm init --upgrade
    # Remove existing helm repositories

    helm repo remove local
    helm repo remove stable
    helm init --upgrade
    sleep 10
    # Start helm local repository in background
    nohup helm serve > helm_serve.log &
    sleep 10
    helm repo add local http://127.0.0.1:8879
    helm repo list
    kubectl taint nodes $(hostname) node-role.kubernetes.io/master- 
}
function install_go() {
   cd $HOME
   sudo rm go1.14.4.linux-amd64.tar.gz
   sudo rm -rf $HOME/go
   sudo rm -rf /usr/local/go
   wget https://dl.google.com/go/go1.14.4.linux-amd64.tar.gz
   sudo tar -C /usr/local -zxvf go1.14.4.linux-amd64.tar.gz
   mkdir -p ~/go/{bin,pkg,src}
   echo 'export GOPATH=$HOME/go' >> ~/.bashrc
   echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
   echo 'export PATH=$PATH:$GOPATH/bin' >> ~/.bashrc
   source ~/.bashrc
   go version
}



echo "Installing Docker"
install_docker
echo "Installing k8s"
install_k8s
echo "Installing helm"
install_helm
echo "Installing go"
install_go
```
