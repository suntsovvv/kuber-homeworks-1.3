# Домашнее задание к занятию «Запуск приложений в K8S»



------


### Задание 1. Создать Deployment и обеспечить доступ к репликам приложения из другого Pod

1. Создать Deployment приложения, состоящего из двух контейнеров — nginx и multitool. Решить возникшую ошибку.  
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
3. После запуска увеличить количество реплик работающего приложения до 2.
4. Продемонстрировать количество подов до и после масштабирования.
5. Создать Service, который обеспечит доступ до реплик приложений из п.1.
6. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложений из п.1.

------

### Задание 2. Создать Deployment и обеспечить старт основного контейнера при выполнении условий

1. Создать Deployment приложения nginx и обеспечить старт контейнера только после того, как будет запущен сервис этого приложения.
2. Убедиться, что nginx не стартует. В качестве Init-контейнера взять busybox.
3. Создать и запустить Service. Убедиться, что Init запустился.
4. Продемонстрировать состояние пода до и после запуска сервиса.

------

### Правила приема работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать файлы манифестов и ссылки на них в файле README.md.

------
