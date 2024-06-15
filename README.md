How to deploy nodejs application on kubernetes.

Step1 : Creating Node.js application

Install npm package:

$ sudo  apt-get update  
$ sudo apt install npm

Initialize a Node.js project (using npm):   
$ npm init

Create a Simple REST API: Install Express.js: 
$ npm install express

create a Server.js file: 
$vim server.js 

run node.js application: 
$ node server.js

You will get output as Hello world to check that use command:
$ curl localhost:8080/devops

Step2 : Installing Docker, create file and image

Installed Docker on the EC2 machine using below commands.

$ sudo apt update

$ sudo apt install apt-transport-https ca-certificates curl software-properties-common

$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add –

$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"

$ apt-cache policy docker-ce

$ sudo apt install docker-ce

$ sudo usermod -aG docker ${USER}

Under app Directory create docker file:

$ vim dockerfile

Create .dockerignore file

Building a image using command: 

$ docker build -t express:v0.1.0 -f dockerfile .

Pushing the docker image to docker hub: 

$ docker tag 56c3d10a57a6 manasack/manasanodejsapp:v3

$ docker push manasack/manasanodejsapp:v3

Step3: Deploying the image to minikube or Kubernetes

Installing kubectl:

$ curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

$ curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"

$ echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check

$ sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

Installing EKSCTL:

$ ARCH=amd64

$ PLATFORM=$(uname -s)_$ARCH

$ curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"

$ curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check

$ tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz

$ sudo mv /tmp/eksctl /usr/local/bin

Installing Python:
$ sudo apt-get install python3-pip

Installing AWS CLI:

$ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

$ sudo apt install unzip

$ unzip awscliv2.zip

$ sudo ./aws/install

We need to create a file  $  vim cluster.yaml
Before creating cluster we need to attach IAM role to EC2 machine with administrator access.
Creating cluster: 

$ Eksctl create cluster -f cluster.yaml

In folder k8s creating file express.yaml
Then run command: 

$ kubectl apply -f k8s/express.yaml

for output: 
$ kubectl -n staging port-forward svc/express 8080:8080

Step 4: Integrating Prometheus and Grafana using helm-charts.

Installing helm:

$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3

$ chmod 700 get_helm.sh

$ ./get_helm.sh

"Before installing Prometheus and Grafana we need to create CSI driver for volume purpose"

PROMETHEUS: 
Execute the command:

$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

$ helm repo update

$ helm install prometheus prometheus-community/prometheus -f prometheus.yaml -n staging

Let’s run command: 
$ kubectl edit svc prometheus-server -n staging 

#The above command will help us to edit service, after executing this command file will open up so there, we can edit type as type: LoadBalancer and save the file.
Again, if we check “kubectl get svc -n staging”, we will be able to see external IP where we can access that in the browser to view Prometheus page.
Ex: aec042f5dcb3a49e687e8663e327d5a0-1969271481.us-east-1.elb.amazonaws.com

Grafana using helm chart:
Execute the command:

$ helm repo add grafana https://grafana.github.io/helm-charts

Then create grafana.yaml 

$ helm install grafana grafana/grafana -f grafana.yaml -n staging

Integrating Grafana with Prometheus:
Then go to Data sources and select Prometheus and configure it with using Prometheus server details which we just created>>>save and test

Then go to Grafana.com and navigate to dashboards (At the end) where we can fetch code for readymade dashboards >>>> then copy the code
 Then come back to our Grafana Dashboards and select import dashboard and paste the code and load it.
Select Prometheus as source data and then click on import.









