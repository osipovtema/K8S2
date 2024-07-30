# Домашнее задание к занятию «Запуск приложений в K8S»

### Цель задания

В тестовой среде для работы с Kubernetes, установленной в предыдущем ДЗ, необходимо развернуть Deployment с приложением, состоящим из нескольких контейнеров, и масштабировать его.

------

### Задание 1. Создать Deployment и обеспечить доступ к репликам приложения из другого Pod

1. Создать Deployment приложения, состоящего из двух контейнеров — nginx и multitool. Решить возникшую ошибку.  
- Решение [deployment.yml#L27-L31](https://github.com/kibernetiq/netology_k8s/blob/kuber-hw-1-3/deployment.yml#L27-L31)
2. После запуска увеличить количество реплик работающего приложения до 2.
3. Продемонстрировать количество подов до и после масштабирования.
```
yura@ubuntu-test:~/kuber-hw-1-3$ kubectl -n netology get pod
NAME                               READY   STATUS    RESTARTS   AGE
myapp-deployment-6b5df4bcf-8pktk   2/2     Running   0          82s

yura@ubuntu-test:~/kuber-hw-1-3$ vim deployment.yaml

yura@ubuntu-test:~/kuber-hw-1-3$ kubectl -n netology apply -f deployment.yaml
deployment.apps/myapp-deployment configured

yura@ubuntu-test:~/kuber-hw-1-3$ kubectl -n netology get pod
NAME                               READY   STATUS    RESTARTS   AGE
myapp-deployment-6b5df4bcf-8pktk   2/2     Running   0          22m
myapp-deployment-6b5df4bcf-965jt   2/2     Running   0          4s
```
4. Создать Service, который обеспечит доступ до реплик приложений из п.1.  
- [service.yml](https://github.com/kibernetiq/netology_k8s/blob/kuber-hw-1-3/service.yml)
5. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложений из п.1.
```
yura@ubuntu-test:~/kuber-hw-1-3$ kubectl -n netology apply -f service.yaml
service/myapp-service created

yura@ubuntu-test:~/kuber-hw-1-3$ kubectl -n netology get svc
NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE
myapp-service   ClusterIP   10.152.183.198   <none>        80/TCP,1180/TCP   5s

yura@ubuntu-test:~/kuber-hw-1-3$ kubectl -n netology run mycurlpod --image=curlimages/curl -i --tty --rm -- sh
~ $ curl myapp-service:80 -I
HTTP/1.1 200 OK
Server: nginx/1.14.2
Date: Mon, 30 Oct 2023 17:29:57 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 04 Dec 2018 14:44:49 GMT
Connection: keep-alive
ETag: "5c0692e1-264"
Accept-Ranges: bytes

~ $ curl myapp-service:1180 -I
HTTP/1.1 200 OK
Server: nginx/1.24.0
Date: Mon, 30 Oct 2023 17:30:05 GMT
Content-Type: text/html
Content-Length: 154
Last-Modified: Mon, 30 Oct 2023 17:21:22 GMT
Connection: keep-alive
ETag: "653fe612-9a"
Accept-Ranges: bytes
```

------

### Задание 2. Создать Deployment и обеспечить старт основного контейнера при выполнении условий

1. Создать Deployment приложения nginx и обеспечить старт контейнера только после того, как будет запущен сервис этого приложения.
- [deployment_with_init.yml](https://github.com/kibernetiq/netology_k8s/blob/kuber-hw-1-3/deployment_with_init.yml)
2. Убедиться, что nginx не стартует. В качестве Init-контейнера взять busybox.
```
yura@Skynet kubernetes % kubectl -n netology create -f deployment_with_init.yml 
deployment.apps/nginx-deployment created

yura@Skynet kubernetes % kubectl -n netology get pod                       
NAME                                READY   STATUS     RESTARTS   AGE
nginx-deployment-747858db59-4xgfd   0/1     Init:0/1   0          5s
```
3. Создать и запустить Service. Убедиться, что Init запустился.
- [service_with_init.yml](https://github.com/kibernetiq/netology_k8s/blob/kuber-hw-1-3/service_with_init.yml)
```
yura@Skynet kubernetes % kubectl -n netology create -f service_with_init.yml   
service/nginx-service created

yura@Skynet kubernetes % kubectl -n netology get svc                        
NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
nginx-service   ClusterIP   10.152.183.123   <none>        80/TCP    6s

yura@Skynet kubernetes % kubectl -n netology get pod
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-747858db59-jztjr   1/1     Running   0          33s
```
4. Продемонстрировать состояние пода до и после запуска сервиса.

------