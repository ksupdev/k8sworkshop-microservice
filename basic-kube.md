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

- สร้าง pods ของ nginx ``การสร้าง pod เราจะสร้างผ่าน deployment file `` (kind: Deployment)
```powershell
kubectl apply -f 01-deployment.yml
deployment.apps/nginx created
```
- ทำการตรวจสอบ pod

```powershell
kubectl get pod -n basic-k8s
NAME                     READY   STATUS    RESTARTS   AGE
nginx-86469cbbfd-s64v8   1/1     Running   1          17h
```

> readinessProbe : ใช้สำหรับตรวจสอบสถานะว่า``พร้อมใช้งานแล้วหรือยัง`` (01-deployment.yml)
> livenessProbe : ใช้สำหรับตรวจสอบสถานะว่า``ยังใช้งานได้อยู่หรือเปล่า`` (01-deployment.yml)

- ทำการแก้ไข 01-deployment.yml ให้มีค่า `` replicas :2 `` หรือก็คือการกำหนด ให้มีการสร้าง pods เป็นจำนวน 2 ตัวนั้นเอง
``` powershell
% kubectl get pods -n basic-k8s
NAME                     READY   STATUS    RESTARTS   AGE
nginx-86469cbbfd-s64v8   1/1     Running   1          18h
nginx-86469cbbfd-t55dt   0/1     Running   0          37s
```

- ทำการสร้าง service และ check service ที่เราสร้าง โดยเราจะต้องมีการกำหนดด้วยว่า service จะมีการ connect ไปที่ pods ชุดไหน `` คำว่า pods ชุดไหนหมายความว่า pods อาจจะมีการกำหนด replica ได้หลายตัวจึงทำให้เกิดหลาย pod ใน deployment นั้นได้ `` (kind: Service)
```powershell
kubectl apply -f 02-service.yml 
service/nginx created

---- check service ----

kubectl get svc -n basic-k8s
NAME    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
nginx   ClusterIP   10.99.131.165   <none>        8080/TCP,8443/TCP   54s

```

- ทำการสร้าง Ingress และทำการ check (kind: Ingress)
> ingress จะเป็นตัวที่สามารถทำให้เรา access service จากภายนอก cluster ได้
```powershell

% kubectl apply -f 03-ingress.yml 
Warning: extensions/v1beta1 Ingress is deprecated in v1.14+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
ingress.extensions/ingress created
--- Check ingress ---
% kubectl get ing -n basic-k8s
Warning: extensions/v1beta1 Ingress is deprecated in v1.14+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
NAME      CLASS    HOSTS                        ADDRESS   PORTS   AGE
ingress   <none>   kubernetes.docker.internal             80      39s
```

ใน file 03-ingress.yml นั้นจะมีการกำหนด host ``kubernetes.docker.internal`` โดย host ตัวนี้จะมีการ auto setup ในเครื่องเราตั้งแต่ตอนติดตั้ง minikub แล้ว โดยเราสามารถเข้าไปดูที่ ``cat /private/etc/hosts`` ซึ่งจะกำหนดเป็น ``127.0.0.1 kubernetes.docker.internal`` นั้นเอง

```yml
spec:
  rules:
  - host: kubernetes.docker.internal
    http:
      paths:
      - path: /
        backend:
          serviceName: nginx
          servicePort: 8080

--- result of cat /private/etc/hosts ----

karoon@Nuttakorns-MacBook-Pro 02-run-nginx-with-k8s % cat /private/etc/hosts
##
# Host Database
#
# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.
##
127.0.0.1       localhost
255.255.255.255 broadcasthost
::1             localhost
# Added by Docker Desktop
# To allow the same kube context to work on the host and the container:
127.0.0.1 kubernetes.docker.internal
# End of section

```

- เราสามารถ test โดยใช้
```curl
curl -X GET "http://kubernetes.docker.internal"
```


- ทำการ ทดสอบ delete namespace ทั้งหมด ซึ่งจะเป็นการ delete ทุกอย่างที่เราได้ set ไว้ทั้งหมดเลย
```powershell
kubectl delete ns basic-k8s
namespace "basic-k8s" deleted
```

