# Лабораторная работа №3 "Сертификаты и "секреты" в Minikube, безопасное хранение данных."

University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)

Year: 2023/2024

Group: K4112с

Author: Moskovsky Anton Aleksandrovich

Lab: Lab3

Date of create: 12.12.2024

Date of finished: 12.12.2024

**1. Развернуть minikube cluster**
```
$ minikube start
```
**2. Манифест файла для развертывания deployment-react**

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: react-env
data:
  react_app_username: "moskovsky"
  react_app_company_name: "itmo"

---

apiVersion: apps/v1
kind: ReplicaSet                                            
metadata:
  name: react-repset                   
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
              valueFrom:
                configMapKeyRef:
                  name: react-env
                  key: react_app_username
            - name: REACT_APP_COMPANY_NAME
              valueFrom:
                configMapKeyRef:
                  name: react-env
                  key: react_app_company_name

---

apiVersion: v1
kind: Service
metadata:
  name: react-service
spec:
  selector:
    app: react
  type: ClusterIP
  ports:
    - port: 3010
      name: react-port
      targetPort: react-port
      protocol: TCP

---

apiVersion: v1
kind: Secret
metadata:
  name: react-tls
type: kubernetes.io/tls
data:
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUREekNDQWZlZ0F3SUJBZ0lVVXp6ek9TNWl6Nlg4TGZybHpNV3h0NTRCa2xRd0RRWUpLb1pJaHZjTkFRRUwKQlFBd0Z6RVZNQk1HQTFVRUF3d01iVzl6YTI5MmMydDVMbTFsTUI0WERUSTBNVEl4TWpBME5UVTBOMW9YRFRJMQpNVEl4TWpBME5UVTBOMW93RnpFVk1CTUdBMVVFQXd3TWJXOXphMjkyYzJ0NUxtMWxNSUlCSWpBTkJna3Foa2lHCjl3MEJBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUFyci94VW9aaURWaytqRmdKcXZ3NllteGoyMllKTWkxMlJVVi8KRTNSYm45WTY3ZXVFV1VlRktIUi95WElPTm9vUzN4eXNXclpzbGZNMzNJRUlFcHhOdjAxa29yakgvS0RwM2I1UQp5MksyZ2E3dk1WY0M2ckZpU3d6dkxOWVZuaUIyblVVRzRhR2JtZ0VDOG1OTDRRUXdPL0JFUi9FSm5EM0dUYlAvCnhYajRQKzRZaDc5UVNRREdzWHB6eTBJOXgvV1JBRHpjcktUYUtYMHNIc25GeVo5T2JFT3hGbTJsTTVDT1d6TTAKVWIrLzVCcjh2L1UrazVqU2FnSnErWVp1dDJ5alFWbFBLQjZvOUtvSFJnVXZyMWJNY2EzTmp0V0dXcWw1VlVwNwpYdE1WUGxJZVQyS0liZ2JPN0VqYlU5R251NWc3VlFZSmJqYitCVElUNFMwZkNXMkNCUUlEQVFBQm8xTXdVVEFkCkJnTlZIUTRFRmdRVUR4ZEZpM1JLeEE5M0Q0K3g2Y3RoRjBOM1U2a3dId1lEVlIwakJCZ3dGb0FVRHhkRmkzUksKeEE5M0Q0K3g2Y3RoRjBOM1U2a3dEd1lEVlIwVEFRSC9CQVV3QXdFQi96QU5CZ2txaGtpRzl3MEJBUXNGQUFPQwpBUUVBRUhhNU5sNnZMcU9QQlZvSUVkQWorejZjcnBHUEcwS0JhZHNoZ0NXL0pxYURxTTExNitRNHE0Q0d1anlvCmtVdFJIYUxLbms5QXl0YjNleEg1WFdvZzVaSm8rZFlmWVZwTFRlUWltVWpQb3c4WUZhTGhLZDlrbTNqQWduT2YKWVFrSitmb1JuKzMxM1M4ZXFWbm5SWTJkVjg5UXNsQ1Aybnl1dVRPdUt3a3RVSDFBcTRHQjdGUFZkdFFIVlBRWAoxSGw5VlZOVWgzc1BZK05IUTBaQzI5S1lRQ3lkVnVCTTRLUXQ0RmZRZjhrS2RzZ2dTajJtdVRZMUh2RElYR2lSClFhMnM1R0p6VjlFd1JCT29KZmcrY0JRWjRRUTZ0cWloY281L1diSERZbUpjQlJjQm9uYmFKRkExbmxnQ08yaTIKenF6NDh2R0xLbmVHU3ZySlhBM3I4bzBFUnc9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
  tls.key: LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JSUV2QUlCQURBTkJna3Foa2lHOXcwQkFRRUZBQVNDQktZd2dnU2lBZ0VBQW9JQkFRQ3V2L0ZTaG1JTldUNk0KV0FtcS9EcGliR1BiWmdreUxYWkZSWDhUZEZ1ZjFqcnQ2NFJaUjRVb2RIL0pjZzQyaWhMZkhLeGF0bXlWOHpmYwpnUWdTbkUyL1RXU2l1TWY4b09uZHZsRExZcmFCcnU4eFZ3THFzV0pMRE84czFoV2VJSGFkUlFiaG9adWFBUUx5ClkwdmhCREE3OEVSSDhRbWNQY1pOcy8vRmVQZy83aGlIdjFCSkFNYXhlblBMUWozSDlaRUFQTnlzcE5vcGZTd2UKeWNYSm4wNXNRN0VXYmFVemtJNWJNelJSdjcva0d2eS85VDZUbU5KcUFtcjVobTYzYktOQldVOG9IcWowcWdkRwpCUyt2VnN4eHJjMk8xWVphcVhsVlNudGUweFUrVWg1UFlvaHVCczdzU050VDBhZTdtRHRWQmdsdU52NEZNaFBoCkxSOEpiWUlGQWdNQkFBRUNnZ0VBSTdGY3p5aFhtclpoeWpTcE5OMXo1MnFRTXQzeWZ1YytRd1BnNHM1ZmNKUkgKVVJWTDRSaDBvRUM3WVNBRXV5c1VrN0c5bW9Hc1NDeDNlbmg0ZDZTcFZLdXdKSFJ0bExJaFVvTnU2VHZ1WHlxbAovSVB6T3BDa3JRT2xUcGtqclRxZ1A1czd2cFpOdS9UODd4bE5CRmJncXoxMkZPT3N2TmI1VENHNTJsSE5FdkZ1CmRIcTU2SzhRTjhLM3BrL2ZpWU12b1RzWUFYT0FRNVA4cS8vODBTYXJiUSsxWGg1M1AzdmZzUytlSlg0Vm9sUFUKMTlrbWpnbnNjUzZ0clJLWENlL254VGZUbnRidlIwZ1BJbnlpMXgxcHF3dnZOUDlJNmYyM01KMDhqN3YwcVc4NQpLSXdUd2FzMEpmMStsckQzYlFEcEsvNHRONFNsMU55cld2bkFYUjJtdVFLQmdRRFdDcVZrbExCdE50OGtJTWFBCk9YZ1FmSlJLZmtyc2ViSEZoMVdBSXNMMjJDYnJiV25DWkFQVFd2K21kNVdNbjZRcncvckdSMXczMDVQNXBFZ3MKbjB5QjBlcXkxM3lSLzFKejI2a0dsQ1dSREtaNnNhbWlCa1NuREt6Ny9TSzd0dlhoQlB0bDZCWlFUblpsQjBuNApYQlc2L2FMRyswQ1czVUVVcWpvZjFOVzdkd0tCZ1FEUkFYOFU2T00xUjVwZHdwYmNvR0I4eTlJQ1dzNVRTeDViCm5Vc3RWNHJTbDV3T2FiZG1kdWZSSUd1UU05RGVGRWZjenpCa3AwS1B2ZldaRkEvY01HVkFncG84a0xDQjNDWVMKNDMrckhub1V5dTVyTmZBU2dveFpKSjFtQ3REUXd5VkRycjZRNmhoUXNlamRhaHJkV1U2WUhXb3lucThCTVp2Wgp4cGRKNDVUVll3S0JnR2Y5MmllSStrTEZzeHBaZGpmY05CSkdoTUhBcEdSS0orM2hkOC8razV6Y25lUXFUNFRyCmxOUStWUkVxN3BUWkJ5bGdXVm0yVi82anBEUlk5ZHdBTldxcGM0OGFsT0pXRzFoQTg5bEhadzBYQ1ZkNU5BS04KYXhPQ1hCVStBbjhUUUZqb1U4QktSM1VTK2dEUnpzV0U3K1hlenhSQUJEeUlHTk9TZFJUOEVpKzNBb0dBQTFCQgp3b1FhcmdxUGtQTDN6MUdmbGZycFBtNVFIUlB3ekVVSEh4WG5Ob1Yrek4reUw2YXM4Q3pTWjd2YWtOckRkT1czCi83Q0RKcUk5VllyeTRXdkcveW5TNWlqcEUzWDVDSTJneFlhN0tyODQzbXhCZlJtaXZmc05uOE9HSWZrbUN4ZW4KSDhjR1Vha3dadW82dU0ya3FGYTNDMHhtdTk4Y2VHeGtrNkJQQ0w4Q2dZQjZuYlVhMVRORDNQNGs4ZURveGl5bwpaMU5pdFFPcE5BemxPNHFHN3BOcjZiNWZJYzVydWd4QzQ4S1NYYUxITVpkaVJLS2dqNXBhRjc2L2d0a3VZSXFjCmdHVEt1OGtQUkZEMzhpdW4yVitGRGp2VVNIalkrcC81bTNuaEQ0ZVVlNXdhZGRnMjlvSVpqY01NTEJGVjdNY3MKbWxkYU9pOFpqaE1FcnBDVW11UjdJdz09Ci0tLS0tRU5EIFBSSVZBVEUgS0VZLS0tLS0K

---

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: react-ingress
spec:
  tls:
    - secretName: react-tls
      hosts:
        - moskovsky.me
  rules:
    - host: moskovsky.me
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: react-service
                port:
                  number: 3010

```


**3. Привязывание манифеста**
```
$ kubectl apply -f cert.yaml
```
**4. Включение Ingress**
```
$ minikube addons enable ingress
```

**5. Создание сертификата**
```
$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=moskovsky.me"
```
**6. Вывести ключи**
```
$ cat tls.crt | base64
$ cat tls.key | base64

```

**7. Изменить вместо localhost добавить свое название**
```
$ sudo nano /etc/hosts
```

**8. Обновить манифест**
```
$ kubectl apply -f cert.yaml
```

**9. Запуск tunnel**
```
$ minikube tunnel
```

![result_lab_3](https://github.com/user-attachments/assets/a50e3dca-edf2-4158-a22b-00fc3acc8885)
![result_cert](https://github.com/user-attachments/assets/f194688d-449b-48ef-895c-5f2e4d492761)
