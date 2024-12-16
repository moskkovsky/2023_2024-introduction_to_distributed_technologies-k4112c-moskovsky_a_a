# Лабораторная работа №4 "Сети связи в Minikube, CNI и CoreDNS"

University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)

Year: 2023/2024

Group: K4112с

Author: Moskovsky Anton Aleksandrovich

Lab: Lab4

Date of create: 15.12.2024

Date of finished: 

**1. Установка плагина и режима работы Multi-Node Clusters**
```
$ minikube start --network-plugin=cni --cni=calico --nodes 2 -p multinode-demo
```
**2. Проверка работы Calico**

```
kubectl get pods -l k8s-app=calico-node -A
kubectl get nodes
```
![image](https://github.com/user-attachments/assets/406b19c6-5cb3-49e5-9ed1-88bdc2e7872d)


**3. Пометка нод** 

--overwrite=true - если уже помечали nods до этого и хотите rename

```
$ kubectl label nodes multinode-m02 rack=1 --overwrite=true
$ kubectl label nodes multinode rack=0 --overwrite=true 
```
![image](https://github.com/user-attachments/assets/6eebac28-b55a-4ca4-9f68-175be2e31175)


**4. Манифест для Calico**

```
apiVersion: crd.projectcalico.org/v1
kind: IPPool
metadata:
  name: rack-0
spec:
  cidr: 192.168.10.0/24
  ipipMode: Always
  natOutgoing: true
  nodeSelector: rack == "0"
---
apiVersion: crd.projectcalico.org/v1
kind: IPPool
metadata:
  name: rack-1
spec:
  cidr: 192.168.20.0/24
  ipipMode: Always
  natOutgoing: true
  nodeSelector: rack == "1"
```

**5. Скачивание конфигурационного файла**

**(перед командой необходимо [скачать конфигурационный файл)](https://github.com/projectcalico/calico/blob/master/manifests/calicoctl.yaml)**
```
$ kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml
```
![image](https://github.com/user-attachments/assets/06461263-5430-4809-a6c1-7f90efe99c70)

**6. Создание deployment**
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment
  labels:
    app: lab4
spec:
  replicas: 2
  selector: 
    matchLabels:
      app: lab4
  template:
    metadata:
      labels:
        app: lab4
    spec:
      containers:
      - name: lab4
        image: ifilyaninitmo/itdt-contained-frontend:master
        env:
        - name: REACT_APP_USERNAME
          value: moskovsky
        - name: REACT_APP_COMPANY_NAME
          value: itmo
        ports:
        - containerPort:  3000

```

**7. Создание service**
```
apiVersion: v1
kind: Service
metadata:
  name: service-lab4
spec:
  type: NodePort
  selector:
    app: lab4
  ports:
    - port: 3000
      protocol: TCP
      name: http
```

**8. Обновление deployment и service**
```
$ kubectl apply -f deployment.yaml
$ kubectl apply -f service.yaml
```

**9. Вход в веб браузер**
```
localhost:3000
```

![image](https://github.com/user-attachments/assets/709596e3-2859-4bff-9348-694cc1d3d93f)

![image](https://github.com/user-attachments/assets/e597cfda-3318-4e70-9d1b-05487586340c)


**10. Узнать информацию о FQDN имена подов**
```
$ kubectl get pods -o wide
NAME                         READY   STATUS    RESTARTS   AGE   IP               NODE            NOMINATED NODE   READINESS GATES
deployment-d6b47766f-f9vr7   1/1     Running   0          16m   10.244.117.199   multinode       <none>           <none>
deployment-d6b47766f-g9brr   1/1     Running   0          16m   10.244.169.3     multinode-m02   <none>           <none>


$ kubectl exec deployment-d6b47766f-g9brr nslookup 10.244.169.3 
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
Server:		10.96.0.10
Address:	10.96.0.10:53

3.169.244.10.in-addr.arpa	name = 10-244-169-3.service-lab4.default.svc.cluster.local


$ kubectl exec deployment-d6b47766f-f9vr7 nslookup 10.244.117.199
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
Server:		10.96.0.10
Address:	10.96.0.10:53

199.117.244.10.in-addr.arpa	name = 10-244-117-199.service-lab4.default.svc.cluster.local
```

![image](https://github.com/user-attachments/assets/e6a5140e-bad2-466d-af22-7053a912fe44)


**11. Пинг подов**
```
$ kubectl exec -it deployment-d6b47766f-f9vr7  sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
/frontend # ping 10-244-169-3.service-lab4.default.svc.cluster.local
PING 10-244-169-3.service-lab4.default.svc.cluster.local (10.244.169.3): 56 data bytes
64 bytes from 10.244.169.3: seq=0 ttl=62 time=0.454 ms
64 bytes from 10.244.169.3: seq=1 ttl=62 time=0.189 ms
64 bytes from 10.244.169.3: seq=2 ttl=62 time=0.209 ms
64 bytes from 10.244.169.3: seq=3 ttl=62 time=0.471 ms
64 bytes from 10.244.169.3: seq=4 ttl=62 time=0.506 ms
64 bytes from 10.244.169.3: seq=5 ttl=62 time=0.293 ms
64 bytes from 10.244.169.3: seq=6 ttl=62 time=0.454 ms
64 bytes from 10.244.169.3: seq=7 ttl=62 time=0.143 ms

$ kubectl exec -it deployment-d6b47766f-g9brr sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
/frontend # ping 10-244-117-199.service-lab4.default.svc.cluster.local
PING 10-244-117-199.service-lab4.default.svc.cluster.local (10.244.117.199): 56 data bytes
64 bytes from 10.244.117.199: seq=0 ttl=62 time=0.330 ms
64 bytes from 10.244.117.199: seq=1 ttl=62 time=0.545 ms
64 bytes from 10.244.117.199: seq=2 ttl=62 time=9.079 ms
64 bytes from 10.244.117.199: seq=3 ttl=62 time=0.312 ms
64 bytes from 10.244.117.199: seq=4 ttl=62 time=0.121 ms
64 bytes from 10.244.117.199: seq=5 ttl=62 time=0.273 ms
```
![image](https://github.com/user-attachments/assets/87a23847-8f90-43a3-b86a-657d43866469)

![image](https://github.com/user-attachments/assets/cd175231-076e-4842-8cdf-f6f56c988f64)



