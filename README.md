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

