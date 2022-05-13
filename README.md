# CICD-Pipeline 
Pipeline CICD pour une application java à déployer sur un cluster kubernetes à l'aide de Jenkins. 

# Step 1 : install & configur  (Docker, k8s, Jenkins, Sonarqube, Nexus, Helm)

1.Creat 5 VMs in google cloud platform 

  1.1. VM jenkins   
  1.2. VM nexus   
  1.3. VM sonarqube   
  1.4. VM k8s MASTER   
  1.5. VM k8s NODE   

# JENKINGS INSTALATION 
wget https://updates.jenkins-ci.org/download/war/2.162/jenkins.war ( installs 2.162 version, if you want any other version to be installed visit https://updates.jenkins-ci.org/download/war/ download particular version )
java -jar jenkins.war ( default runs on 8080 port )
java -jar jenkins.war --httpPort=5000 ( if you want run on any other port use this, in my case its 5000 port )
nohup java -jar jenkins.jar & ( to run jenkins process in background )

# DOCKER INSTALTION 
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-key fingerprint 0EBFCD88
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io ( to install latest version )
sudo docker run hello-world

#INSTALTION OF SONARCUBE 
create sonarqube through docker then use below command
docker run -d -p 9000:9000 sonarqube:lts

# INSTALATION OF K8S 
Make sure docker installed in master and nodes

1. Execute below commands in both master and node
    sudo apt-get update && sudo apt-get install -y apt-transport-https curl
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
    cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
    deb https://apt.kubernetes.io/ kubernetes-xenial main
    EOF
    sudo apt-get update
    sudo apt-get install -y kubelet kubeadm kubectl
    sudo apt-mark hold kubelet kubeadm kubectl


2. Execute below commands in master
    after executing this command you will get node's joining command, copy and paste it somewhere

    kubeadm init --pod-network-cidr=10.244.0.0/16 ( if you have forget to do then use kubeadm token create --print-join-command )
    export KUBECONFIG=/etc/kubernetes/admin.conf
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config;mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/canal/rbac.yaml
    kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml
    Execute join command in node, which may look like as mentioned below

   kubeadm join 10.128.0.8:6443 --token q915fe.do2ty6a8ow6qjixt \
   --discovery-token-ca-cert-hash sha256:acd137106e6b763d1ca6b5a4f7c1b1538c2ee8af81e47f9ea3f385c66cd710b3 
   
# NEXUS INSTALTION 
apt-get install wget ( install if you dont have wget )
java -version ( make sure java is installed which should be java 8 or higher version )
wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz
tar -xvf latest-unix.tar.gz
cd nexus-3.35.0-02/bin
./nexus start ( starts the nexus artifactory )
./nexus status ( by this you check the status of nexus artifactory )


# Helm Installation 
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3

chmod 700 get_helm.sh

./get_helm.sh

helm uninstallation
which helm ( to see which folder its installed )
rm -rf /usr/local/bin/helm

#HOW THE PIPELINE WORKS 

1. pull the code from github
2. static code analysis using sonarqube and gradle
3. cheack if the code passes or not.
 
        3.1. if cheack fail send email to director  with link to jenkins and the version of the build
  
        3.2. if cheack successfull move on 
  
4. using dokerfile build the artifact and create docker image 
5. push image to nexus 
6. check for misconfiguration in helm charts using DATREE.IO.
  
        6.1. if cheack fail send email to director with link to jenkins and version of the build 
  
        6.2. if cheack successful move on 

7. helm charts push to nexus 
8. send email to director  to get manual approval for deployement 
9. Deploy on k8s cluster

# Generale information 
Integrating Sonarqube with Jenkins

Creating Docker hosted repository in Nexus and pushing the docker image through Jenkins

Creating Helm hosted repository in Nexus and Pushing the helm charts

Configuring mail server in Jenkins

Configuring PR based trigger in Jenkins

connecting jenkins with kubernetes cluster

configuring mail sever & adding post block in Jenkinsfile for  sending mail 

Identifying the misconfigurations, yaml validation, schema validation using datree & adding the stage in Jenkinsfile   
