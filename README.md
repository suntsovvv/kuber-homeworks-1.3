# Домашнее задание к занятию «Запуск приложений в K8S»



------


### Задание 1. Создать Deployment и обеспечить доступ к репликам приложения из другого Pod


Создал манифест mydeployment.yaml:
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
  labels:
    app: md
spec:
#  replicas: 3
  selector:
    matchLabels:
      app: md
  template:
    metadata:
      labels:
        app: md
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
      - name: multitool
        image: wbitt/network-multitool
        ports:
        - containerPort: 8080
        env:
        - name: HTTP_PORT
          value: "1180"
```
Запустил и проверил:
```bash
user@microk8s:~$ kubectl apply -f mydeployment.yaml 
deployment.apps/my-deployment configured
user@microk8s:~$ kubectl get deployments.apps 
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
my-deployment   1/1     1            1           57m
user@microk8s:~$ kubectl get po
NAME                             READY   STATUS    RESTARTS   AGE
my-deployment-8569b9d87d-xhpfb   2/2     Running   0          7m26s
user@microk8s:~$ 
```
Добавил в манифест параметр   replicas: 2
Запустил и проверил:
```bash
user@microk8s:~$ kubectl apply -f mydeployment.yaml 
deployment.apps/my-deployment configured
user@microk8s:~$ kubectl get deployments.apps 
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
my-deployment   2/2     2            2           59m
user@microk8s:~$ kubectl get po
NAME                             READY   STATUS    RESTARTS   AGE
my-deployment-8569b9d87d-fdbbn   2/2     Running   0          16m
my-deployment-8569b9d87d-xhpfb   2/2     Running   0          25m 
```

5. Создать Service, который обеспечит доступ до реплик приложений из п.1.
Создал манифест md-svc.yaml:
```yml
apiVersion: v1
kind: Service
metadata:
  name: md-svc
spec:
  selector:
    app: md
  ports:
    - protocol: TCP
      name: ngiinx
      port: 80
      targetPort: 80
    - protocol: TCP
      name: multitool
      port: 8080
      targetPort: 1180
```
Запустил и проверил service:
```bash
user@microk8s:~$ kubectl apply -f md-svc.yaml 
service/md-svc created
user@microk8s:~$ kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE
kubernetes   ClusterIP   10.152.183.1     <none>        443/TCP           5d9h
md-svc       ClusterIP   10.152.183.240   <none>        80/TCP,8080/TCP   63s
```
Создал манифест multitool.yaml:
```yml
apiVersion: v1
kind: Pod
metadata:
  name: multitool
spec:
  containers:
    - name: multitool
      image: wbitt/network-multitool
      ports:
        - containerPort: 8080
```
Запустил и проверил:
```bash
user@microk8s:~$ kubectl apply -f multitool.yaml 
pod/multitool created
user@microk8s:~$ kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
multitool                        1/1     Running   0          39s
my-deployment-8569b9d87d-fdbbn   2/2     Running   0          3h50m
my-deployment-8569b9d87d-xhpfb   2/2     Running   0          3h59m
```
Убедиться с помощью `curl`, что из пода есть доступ до приложений:
```bash
user@microk8s:~$ kubectl exec multitool -- curl md-svc:80
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
100   615  100   615    0     0  30148      0 --:--:-- --:--:-- --:--:-- 30750
user@microk8s:~$ kubectl exec multitool -- curl md-svc:8080
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   149  100   149    0     0   122k      0 --:--:-- --:--:-- --:--:--  145k
WBITT Network MultiTool (with NGINX) - my-deployment-8569b9d87d-fdbbn - 10.1.128.238 - HTTP: 1180 , HTTPS: 443 . (Formerly praqma/network-multitool)
user@microk8s:~$
```
------

### Задание 2. Создать Deployment и обеспечить старт основного контейнера при выполнении условий

Создал манифест md-init.yaml:
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: md-init
  labels:
    app: md
spec:
#  replicas: 2
  selector:
    matchLabels:
      app: init-nginx
  template:
    metadata:
      labels:
        app: init-nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
      initContainers:
      - name: busybox
        image: busybox
        command: ['sh', '-c', "until nslookup md-init-svc.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for md-init-svc; sleep 2; done"]
```
Проверяем запуск:
```bash
user@microk8s:~$ kubectl apply -f md-init.yaml 
deployment.apps/md-init created
user@microk8s:~$ kubectl get deployments.apps 
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
md-init         0/1     1            0           7s
my-deployment   2/2     2            2           6h17m
user@microk8s:~$ kubectl get po
NAME                             READY   STATUS     RESTARTS   AGE
md-init-84f8b667c9-dgtq2         0/1     Init:0/1   0          16s
multitool                        1/1     Running    0          88m
my-deployment-8569b9d87d-fdbbn   2/2     Running    0          5h18m
my-deployment-8569b9d87d-xhpfb   2/2     Running    0          5h27m
```
Видим статус Init:0/1.
Создал манифест md-init-svc.yaml:
```yml
apiVersion: v1
kind: Service
metadata:
  name: md-init-svc
spec:
  selector:
    app: init-nginx
  ports:
    - name: nginx
      port: 80
```
Запустил сервис и проверил состояние:
```bash
user@microk8s:~$ kubectl apply -f md-init-svc.yaml 
service/md-init-svc created
user@microk8s:~$ kubectl get po
NAME                             READY   STATUS            RESTARTS   AGE
md-init-84f8b667c9-dgtq2         0/1     PodInitializing   0          16m
multitool                        1/1     Running           0          105m
my-deployment-8569b9d87d-fdbbn   2/2     Running           0          5h35m
my-deployment-8569b9d87d-xhpfb   2/2     Running           0          5h44m
user@microk8s:~$ kubectl get po
NAME                             READY   STATUS    RESTARTS   AGE
md-init-84f8b667c9-dgtq2         1/1     Running   0          16m
multitool                        1/1     Running   0          105m
my-deployment-8569b9d87d-fdbbn   2/2     Running   0          5h35m
my-deployment-8569b9d87d-xhpfb   2/2     Running   0          5h44m
```



Ссылка на манифесты:
https://github.com/suntsovvv/kuber-homeworks-1.3

------
