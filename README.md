# minikube-on-centos7-using-kvm

QEMU: https://access.redhat.com/articles/1344173#Q_how-install-virtualization-packages
```
sudo su -
yum install qemu-kvm libvirt libvirt-python libguestfs-tools virt-install -y
systemctl start libvirtd.service && systemctl enable libvirtd.service
```
```
exit
```

QEMU group: https://computingforgeeks.com/use-virt-manager-as-non-root-user/

Check ---
```
sudo getent group | grep libvirt
```
Add --
```
sudo usermod -a -G libvirt $(whoami)
newgrp libvirt
```
Check --
```
id $(whoami)
```
Restart --
```
sudo systemctl restart libvirtd.service
egrep -q 'vmx|svm' /proc/cpuinfo && echo yes || echo no
```
KUBECTL:
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x ./kubectl && sudo mv ./kubectl /usr/local/bin/kubectl
```
Check--
```
kubectl version --client
```
MINIKUBE: 
```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
   && sudo install minikube-linux-amd64 /usr/local/bin/minikube
```
```
minikube config set vm-driver kvm2
minikube config set memory 8192
minikube config set cpus 4
```
```
minikube start
```
```
minikube status
kubectl version
kubectl get pods -A
```
### Minikube Test
```
minikube dashboard
kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.4
kubectl get pods
kubectl expose deployment hello-minikube --type=NodePort --port=8080
minikube service hello-minikube
```
### Helm
```
minikube addons enable helm-tiller
```
```
curl -LO https://get.helm.sh/helm-v2.16.3-linux-amd64.tar.gz
tar -zxvf helm-v2.16.3-linux-amd64.tar.gz
sudo mv linux-amd64/helm /usr/local/bin/
```
### Test Jenkins
```
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Namespace
metadata:
  name: jenkins
EOF
```
```
cat <<EOF | kubectl create -f -
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
EOF
```
```yaml
master:
  servicePort: 8080
  serviceType: NodePort
  nodePort: 32123
  scriptApproval:
    - "method groovy.json.JsonSlurperClassic parseText java.lang.String"
    - "new groovy.json.JsonSlurperClassic"
    - "staticMethod org.codehaus.groovy.runtime.DefaultGroovyMethods leftShift java.util.Map java.util.Map"
    - "staticMethod org.codehaus.groovy.runtime.DefaultGroovyMethods split java.lang.String"
  InstallPlugins:
  - kubernetes:1.7.1   
  - workflow-aggregator:2.5   
  - workflow-job:2.21   
  - credentials-binding:1.16   
  - git:3.9.1   
agent:
  volumes:
    - type: HostPath
      hostPath: /var/run/docker.sock
      mountPath: /var/run/docker.sock

persistence:
  Enabled: true
  StorageClass: jenkins-volume   
  Size: 10Gi

networkPolicy:
  Enabled: false
  ApiVersion: extensions/v1beta1

rbac:
  create: true
  serviceAccountname: default
  apiVersion: v1beta1
#  roleRef: cluster-admin
```
```
helm install --name jenkins --namespace jenkins --values values.yaml stable/jenkins
```
```
printf $(kubectl get secret --namespace jenkins jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
```
```
minikube --namespace=jenkins service jenkins
```
