# Jenkins/Kube/Helm demo

The accompanying presentation can be found on [SlideShare](https://www.slideshare.net/davidcurrie/continuous-delivery-to-kubernetes-with-jenkins-and-helm).

## Pre-reqs

```bash
install minikube
minikube start
minikube addons enable ingress
minikube addons enable registry
install kubectl
install helm
```

## Set up Helm

```bash
helm init 
```

## Deploy Jenkins

```bash
helm install
  --name jenkins
  --namespace jenkins
  --values values.yml
  stable/jenkins

printf $(kubectl get secret --namespace jenkins jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo

minikube service jenkins --namespace=jenkins

jenkins-namespace.yml 
---
apiVersion: v1
kind: Namespace
metadata:
  name: jenkins
  
jenkins-volume.yml 
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: jenkins-volume
  namespace: jenkins
spec:
  storageClassName: jenkins-volume
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 10Gi
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /data/jenkins/
    
values.yml 
Master:
  ServicePort: 8080
  ServiceType: NodePort
  NodePort: 32123
  ScriptApproval:
    - "method groovy.json.JsonSlurperClassic parseText java.lang.String"
    - "new groovy.json.JsonSlurperClassic"
    - "staticMethod org.codehaus.groovy.runtime.DefaultGroovyMethods leftShift java.util.Map java.util.Map"
    - "staticMethod org.codehaus.groovy.runtime.DefaultGroovyMethods split java.lang.String"
  InstallPlugins:
    - kubernetes:1.6.4
    - workflow-aggregator:2.5
    - workflow-job:2.21
    - credentials-binding:1.16
    - git:3.9.1
Agent:
  volumes:
    - type: HostPath
      hostPath: /var/run/docker.sock
      mountPath: /var/run/docker.sock

Persistence:
  Enabled: true
  StorageClass: jenkins-volume
  Size: 10Gi

NetworkPolicy:
  Enabled: false
  ApiVersion: extensions/v1beta1

rbac:
  install: true
  
kubectl apply -f <file_name> .
wget https://storage.googleapis.com/kubernetes-helm/helm-v2.11.0-linux-amd64.tar.gz
tar -zxvf helm-v2.11.0-linux-amd64.tar.gz 
sudo cp  linux-amd64/helm /usr/local/bin/helm
helm init

N.B. 
kubectl -n jenkins create sa jenkins
kubectl create clusterrolebinding jenkins --clusterrole cluster-admin --serviceaccount=jenkins:jenkins

Failed to count the # of live instances on Kubernetes
io.fabric8.kubernetes.client.KubernetesClientException: Failure executing: GET at: https://kubernetes.default/api/v1/namespaces/jenkins/pods?labelSelector=jenkins%3Dslave. Message: Forbidden!Configured service account doesn't have access. Service account may have been revoked. pods is forbidden: User "system:serviceaccount:jenkins:default" cannot list pods in the namespace "jenkins".

!!!! kubectl create clusterrolebinding default --clusterrole cluster-admin --serviceaccount=jenkins:default

kubectl create namespace staging
kubectl create clusterrolebinding default2 --clusterrole cluster-admin --serviceaccount=staging:default
```

## Deploy app

1. Create a pipline job with a fork of this repository as the Git source for the pipeline.
2. Clicking *Build now* will push an image to the registry.
3. Run `helm create hello` and modify `values.yaml` to enable ingress and provide a hostname (`hello.192.168.99.100.nip.io`).
4. Uncomment the helm container and deploy stages in `Jenkinsfile`.
5. Push changes and re-build.

=========================================
$ kubectl get pod --namespace=default

NAME                   READY   STATUS    RESTARTS   AGE

hello-65ff564d-dvhq5   1/1     Running   0          8m

curl hello.192.168.99.100.nip.io

Hello world!

```
Console Output
Started by user admin
 > git rev-parse --is-inside-work-tree # timeout=10
Setting origin to https://github.com/adavarski/K8S-with-jenkins-and-helm
 > git config remote.origin.url https://github.com/adavarski/K8S-with-jenkins-and-helm # timeout=10
Fetching origin...
Fetching upstream changes from origin
 > git --version # timeout=10
 > git config --get remote.origin.url # timeout=10
 > git fetch --tags --progress origin +refs/heads/*:refs/remotes/origin/*
Seen branch in repository origin/master
Seen 1 remote branch
Obtained Jenkinsfile from 01f12143e2076053ec4b79834f87b1a1e7a5a920
Running in Durability level: MAX_SURVIVABILITY
[Pipeline] podTemplate
[Pipeline] {
[Pipeline] node
Still waiting to schedule task
Waiting for next available executor
Agent jenkins-slave-kv2vl-zgz4m is provisioned from template Kubernetes Pod Template
Agent specification [Kubernetes Pod Template] (mypod): 
* [jnlp] jenkins/jnlp-slave:3.10-1(resourceRequestCpu: 200m, resourceRequestMemory: 256Mi, resourceLimitCpu: 200m, resourceLimitMemory: 256Mi)
* [helm] ibmcom/k8s-helm:v2.6.0
* [golang] golang:1.10-alpine
* [docker] docker:18.02

Running on jenkins-slave-kv2vl-zgz4m in /home/jenkins/workspace/helm-chart-hello-k8s_master-PCFNCKW3DJ7SKITZHTS7TADTXS6EIV3C6J42SHFZAJRV5MGPDXHA
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Extract)
[Pipeline] checkout
Cloning the remote Git repository
Cloning with configured refspecs honoured and without tags
Cloning repository https://github.com/adavarski/K8S-with-jenkins-and-helm
 > git init /home/jenkins/workspace/helm-chart-hello-k8s_master-PCFNCKW3DJ7SKITZHTS7TADTXS6EIV3C6J42SHFZAJRV5MGPDXHA # timeout=10
Fetching upstream changes from https://github.com/adavarski/K8S-with-jenkins-and-helm
 > git --version # timeout=10
 > git fetch --no-tags --progress https://github.com/adavarski/K8S-with-jenkins-and-helm +refs/heads/*:refs/remotes/origin/*
 > git config remote.origin.url https://github.com/adavarski/K8S-with-jenkins-and-helm # timeout=10
 > git config --add remote.origin.fetch +refs/heads/*:refs/remotes/origin/* # timeout=10
 > git config remote.origin.url https://github.com/adavarski/K8S-with-jenkins-and-helm # timeout=10
Fetching without tags
Fetching upstream changes from https://github.com/adavarski/K8S-with-jenkins-and-helm
 > git fetch --no-tags --progress https://github.com/adavarski/K8S-with-jenkins-and-helm +refs/heads/*:refs/remotes/origin/*
Checking out Revision 01f12143e2076053ec4b79834f87b1a1e7a5a920 (master)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f 01f12143e2076053ec4b79834f87b1a1e7a5a920
Commit message: "Rename chart.yaml to Chart.yaml"
 > git rev-list --no-walk 8f6f995dd74f87dae7616ace1816514142d3ae26 # timeout=10
[helm-chart-hello-k8s_master-PCFNCKW3DJ7SKITZHTS7TADTXS6EIV3C6J42SHFZAJRV5MGPDXHA] Running shell script
[Pipeline] sh
Cannot contact jenkins-slave-kv2vl-zgz4m: java.lang.InterruptedException
+ git rev-parse --short HEAD
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Build)
[Pipeline] container
[Pipeline] {
[Pipeline] sh
[helm-chart-hello-k8s_master-PCFNCKW3DJ7SKITZHTS7TADTXS6EIV3C6J42SHFZAJRV5MGPDXHA] Running shell script
+ CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .
Cannot contact jenkins-slave-kv2vl-zgz4m: java.lang.InterruptedException
[Pipeline] }
[Pipeline] // container
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Docker)
[Pipeline] container
[Pipeline] {
[helm-chart-hello-k8s_master-PCFNCKW3DJ7SKITZHTS7TADTXS6EIV3C6J42SHFZAJRV5MGPDXHA] Running shell script
[Pipeline] sh
+ awk { print $1 ; exit }
+ getent hosts registry.kube-system
[Pipeline] sh
[helm-chart-hello-k8s_master-PCFNCKW3DJ7SKITZHTS7TADTXS6EIV3C6J42SHFZAJRV5MGPDXHA] Running shell script
+ docker build -t 10.98.235.223:80/hello:01f1214 .
Sending build context to Docker daemon  6.716MB

Step 1/3 : FROM scratch
 ---> 
Step 2/3 : COPY main /
 ---> Using cache
 ---> b66c6e59e8a8
Step 3/3 : CMD ["/main"]
 ---> Using cache
 ---> fd90f37c2dac
Successfully built fd90f37c2dac
Successfully tagged 10.98.235.223:80/hello:01f1214
[helm-chart-hello-k8s_master-PCFNCKW3DJ7SKITZHTS7TADTXS6EIV3C6J42SHFZAJRV5MGPDXHA] Running shell script
[Pipeline] sh
+ docker push 10.98.235.223:80/hello:01f1214
The push refers to repository [10.98.235.223:80/hello]
Cannot contact jenkins-slave-kv2vl-zgz4m: java.lang.InterruptedException
163fc829040e: Preparing
163fc829040e: Layer already exists
01f1214: digest: sha256:80bd049f94f45a2b7c8919bb89b1bc8ec94945e3a22d2cf837ead411fd873995 size: 528
[Pipeline] }
[Pipeline] // container
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Deploy)
[Pipeline] container
[Pipeline] {
[Pipeline] sh
[helm-chart-hello-k8s_master-PCFNCKW3DJ7SKITZHTS7TADTXS6EIV3C6J42SHFZAJRV5MGPDXHA] Running shell script
+ /helm init --client-only --skip-refresh
Creating /home/jenkins/.helm 
Creating /home/jenkins/.helm/repository 
Creating /home/jenkins/.helm/repository/cache 
Creating /home/jenkins/.helm/repository/local 
Creating /home/jenkins/.helm/plugins 
Creating /home/jenkins/.helm/starters 
Creating /home/jenkins/.helm/cache/archive 
Creating /home/jenkins/.helm/repository/repositories.yaml 
$HELM_HOME has been configured at /home/jenkins/.helm.
Not installing Tiller due to 'client-only' flag having been set
Happy Helming!
[Pipeline] sh
[helm-chart-hello-k8s_master-PCFNCKW3DJ7SKITZHTS7TADTXS6EIV3C6J42SHFZAJRV5MGPDXHA] Running shell script
+ /helm upgrade --install --wait --set image.repository=10.98.235.223:80/hello,image.tag=01f1214 hello hello
Release "hello" does not exist. Installing it now.

[Pipeline] End of Pipeline
Finished: SU
```
