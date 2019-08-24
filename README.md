# Binder-deploy

Some helpful scripts for getting [BinderHub](http://binderhub.readthedocs.io/) deployed on Google Cloud

* Use Google Cloud Shell [Google documentation](https://cloud.google.com/shell/docs/starting-cloud-shell) which will give you access to the gcloud command-line tool.
* pip install --user cookiecutter
* export PATH=$PATH:~/.local/bin
* cookiecutter https://github.com/nicain/binder-deploy.git

Here you will need to direct cookiecutter to your google password file.  By default, the template will look in ~/.secret/BinderHub.json

* cd binder-deploy && ./setup.sh && ./deploy.sh

I have been seeing an error that I don't know much about: "Error: could not find a ready tiller pod".  If you see this too, just wait a few seconds, and then re-run ./deploy.sh.  Lastly:

* ./info.sh

The command will tell you the IP address of your BinderHub deployment.  When you are all done:

* ./teardown.sh
* Click on the links that are splashed up, and shut down any remaining resources.


# setup.sh

Configure and deploy Kubernetes cluster.

# deploy.sh

Install and connect jupyterhub/binderhub.

# teardown.sh

Tear down jupyterhub/binderhub and kubernetes cluster.  Splash the URL's to double-check all resources are down.

# Useful notes
* Link to latest helm chart hash
https://jupyterhub.github.io/helm-chart/#development-releases-binderhub

* Get Information
kubectl --namespace=binder logs binder-6d6fccdd74-cv6j7
kubectl --namespace=binder describe pods binder-6d6fccdd74-cv6j7
kubectl get nodes

kubectl get pods --all-namespaces
kubectl --namespace=binder get pod -o wide
gcloud compute ssh gke-dl-blockchain-default-pool-0bea9584-h19r 

* Prep laptop/ client machine for gcloud
conda create -n py2.7-google-cloud python=2.7
conda env list
conda activate py2.7-google-cloud
cd ~/Downloads/google-cloud-sdk
ls
./google-cloud-sdk/install.sh
./install.sh 
source ~/.bash_profile

gcloud init
conda env list
source activate py2.7-google-cloud
gcloud init
gcloud components install kubectl

* Create cluster
gcloud container clusters create   --machine-type n1-standard-2   --num-nodes 2   --zone us-west1-a   --cluster-version latest dl-blockchain

* give admin rights
kubectl create clusterrolebinding cluster-admin-binding   --clusterrole=cluster-admin   --user=xxx@gmail.com

* create nodes
gcloud beta container node-pools create user-pool \
  --machine-type n1-standard-2 \
  --num-nodes 0 \
  --enable-autoscaling \
  --min-nodes 0 \
  --max-nodes 3 \
  --node-labels hub.jupyter.org/node-purpose=user \
  --node-taints hub.jupyter.org_dedicated=user:NoSchedule \
  --zone us-west1-a \
  --cluster dl-blockchain
  
* Tiller account
 kubectl --namespace kube-system create serviceaccount tiller

* Tiller admin rights
kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller

* Get helm
curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get | bash

* Initialize
 helm init --service-account tiller --wait

* secure Tiller
kubectl patch deployment tiller-deploy --namespace=kube-system --type=json --patch='[{"op": "add", "path": "/spec/template/spec/containers/0/command", "value": ["/tiller", "--listen=localhost:44134"]}]'

* Generate secrets/tokens api

mkdir binderhub
cd binderhub

openssl rand -hex 32
openssl rand -hex 32

* Add helm repo
helm repo add jupyterhub https://jupyterhub.github.io/helm-chart
helm repo update

* Install Upgrade helm 
helm install jupyterhub/binderhub --version=0.2.0-f746e50  --name=binder --namespace=binder -f config.yaml

* Get jupyterhub IP address (proxy-public)
kubectl --namespace=binder get svc

* Edit config.yaml with ip address, then upgrade helm
helm upgrade binder jupyterhub/binderhub --version=0.2.0-f746e50 -f secret.yaml -f config.yaml

* Teardown
helm list
helm delete jhub --purge
helm delete binder --purge
kubectl delete namespace binder

* get history
history | cut -c 8- >history.txt

