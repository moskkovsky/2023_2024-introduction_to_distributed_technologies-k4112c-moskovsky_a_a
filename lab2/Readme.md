# Лабораторная работа №2 "Развертывание веб сервиса в Minikube, доступ к веб интерфейсу сервиса. Мониторинг сервиса."

University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)

Year: 2023/2024

Group: K4112

Author: Moskovsky Anton Aleksandrovich

Lab: Lab2

Date of create: 12.12.2024

Date of finished: 12.12.2024

**1. Развернуть minikube cluster**
```
$ minikube start
```
**2. Манифест файла для развертывания deployment-react**

```
apiVersion: apps/v1
kind: Deployment                                            
metadata:
  name: deployment-react                         
spec:
  replicas: 2
  selector:
    matchLabels:
      app: react
  template:
    metadata:
      labels:
        app: react
    spec:                                      
      containers:
        - image: ifilyaninitmo/itdt-contained-frontend:master
          name: react                           
          ports:
          - name: react-port
            containerPort: 3000
          env:
            - name: REACT_APP_USERNAME
              value: 'moskovsky'
            - name: REACT_APP_COMPANY_NAME
              value: 'itmo'
```

**replicas:** 
Количество экземпляров приложения, которые должны быть запущены одновременно

**selector:** 
Определяет, какие поды будут управляться этим деплойментом

**3. Манифест файла для Service**
```
apiVersion: v1
kind: Service
metadata:
  name: front-service
spec:
  selector:
    app: react
  type: NodePort
  ports:
    - port: 3010
      name: react-port
      targetPort: react-port
      protocol: TCP
```
**4. Привязывание манифеста**
```
$ kubectl apply -f service.yaml
```
**5. Перенаправление запрос с Service на pods**
```
$ minikube kubectl -- port-forward service/frontend-service 3010:3010
```

**6. Написать в браузере url**
```
http://localhost:3010
```
**7. Создать новый терминал и найти логи**
```
$ minikube kubectl -- get pods

NAME                                READY   STATUS    RESTARTS      AGE
deployment-react-5d7b589df8-2w8gr   1/1     Running   0             19m
deployment-react-5d7b589df8-dcv9x   1/1     Running   0             19m
vault                               1/1     Running   1 (38m ago)   154m


$ minikube kubectl -- logs deployment-react-5d7b589df8-2w8gr
Builing frontend
Browserslist: caniuse-lite is outdated. Please run:
  npx update-browserslist-db@latest
  Why you should do it regularly: https://github.com/browserslist/update-db#readme
Browserslist: caniuse-lite is outdated. Please run:
  npx update-browserslist-db@latest
  Why you should do it regularly: https://github.com/browserslist/update-db#readme
build finished
Server started on port 3000

$ minikube kubectl -- logs deployment-react-5d7b589df8-dcv9x
Builing frontend
Browserslist: caniuse-lite is outdated. Please run:
  npx update-browserslist-db@latest
  Why you should do it regularly: https://github.com/browserslist/update-db#readme
Browserslist: caniuse-lite is outdated. Please run:
  npx update-browserslist-db@latest
  Why you should do it regularly: https://github.com/browserslist/update-db#readme
build finished
Server started on port 3000
```
![image](https://github.com/user-attachments/assets/7d806dc8-d4f5-44d9-be45-b465dfd5b3bb)

![image](https://github.com/user-attachments/assets/919ec7cf-47c6-4c9d-98f2-143394e18f04)

