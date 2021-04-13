### Note kubes

### file 01-deployment.yml
เราจะทำการ deploy application to cluster ของ Kube

- สร้าง namespace
```powershell
$ kubectl apply -f 00-namespace.yml
namespace/basic-k8s created
```

- ตรวจสอบ namespace ที่เราได้สร้างไว้ นั้นก็คือ ``basic-k8s``

```powershell
% kubectl get ns
NAME              STATUS   AGE
basic-k8s         Active   17h
default           Active   58d
ingress-nginx     Active   17h
kube-node-lease   Active   58d
kube-public       Active   58d
kube-system       Active   58d
```