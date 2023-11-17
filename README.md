# Kubernetes-RBAC-OS-user
Kubernetes RBAC limits the access from OS user

Create an OS user  
```
useradd test
```
Switch to test user 
```
su - test
```
Create CSR
```
openssl genrsa -out ~/test.key 2048
openssl req -new -key ~/test.key -out ~/test.csr -subj "/CN=test/O=group1"
```
Approve CSR
```
sudo openssl x509 -req -in ~/test.csr \
-CA /etc/kubernetes/pki/ca.crt \
-CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out ~/test.crt -days 365
```
Create a kuberntes config file for the user
```
touch ~/test.kube.conf 
export KUBECONFIG=~/test.kube.conf
kubectl config set-cluster test --server=https://${cluster-address}:6443 --certificate-authority=/etc/kubernetes/pki/ca.crt
kubectl config set-credentials test --client-certificate=test.crt --namespace=nsrbac --client-key=test.key
kubectl config set-context test --cluster=test --user=test
#Please check changes of ~/test.kube.conf after each above commands.
```
Switch context to test
```
kubectl config use-context test 
```
Switch to admin user.  
create role
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: nsrbac
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```  
Create role binding
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: rd-read-pods
  namespace: nsrbac
subjects:
- kind: User
  name: test
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```
Test RBAC to limit the access of OS user test.    
SSH login to KUBE CONTROL with OS user test.  
```
export KUBECONFIG=~/test.kube.conf
kubectl config view
kubectl get pods -n nsrbac
```

References:  
https://discuss.kubernetes.io/t/how-to-create-user-in-kubernetes-cluster-and-give-it-access/9101/4  
https://www.youtube.com/watch?v=jvhKOAyD8S8&list=PLHq1uqvAteVvUEdqaBeMK2awVThNujwMd&index=14  
