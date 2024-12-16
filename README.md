#«Как работает сеть в K8s» - Абрамов Сергей 

### Цель задания

Настроить сетевую политику доступа к подам.

### Чеклист готовности к домашнему заданию

1. Кластер K8s с установленным сетевым плагином Calico.

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Документация Calico](https://www.tigera.io/project-calico/).
2. [Network Policy](https://kubernetes.io/docs/concepts/services-networking/network-policies/).
3. [About Network Policy](https://docs.projectcalico.org/about/about-network-policy).

-----

### Задание 1. Создать сетевую политику или несколько политик для обеспечения доступа

1. Создать deployment'ы приложений frontend, backend и cache и соответсвующие сервисы.
2. В качестве образа использовать network-multitool.
3. Разместить поды в namespace App.
4. Создать политики, чтобы обеспечить доступ frontend -> backend -> cache. Другие виды подключений должны быть запрещены.
5. Продемонстрировать, что трафик разрешён и запрещён.


### Решение

Создаем namespace:

```
serg@k8snode:~/git/K8s-13$ kubectl create namespace app
namespace/app created
serg@k8snode:~/git/K8s-13$ kubectl get namespace
NAME                     STATUS   AGE
app                      Active   12s
default                  Active   39d
ingress                  Active   31d
kube-node-lease          Active   39d
kube-public              Active   39d
kube-system              Active   39d
nfs-server-provisioner   Active   24d

```

Создаем deployment и svc:

[frontend.yaml](https://github.com/smabramov/K8s-13/blob/413dfab4a668de15f3b6f1046f2ba19933b756bf/code/frontend.yaml)

[backend.yaml](https://github.com/smabramov/K8s-13/blob/413dfab4a668de15f3b6f1046f2ba19933b756bf/code/backend.yaml)

[cache.yaml](https://github.com/smabramov/K8s-13/blob/413dfab4a668de15f3b6f1046f2ba19933b756bf/code/cache.yaml)

```
serg@k8snode:~/git/K8s-13/code$ kubectl apply -f frontend.yaml
deployment.apps/frontend created
service/frontend created
serg@k8snode:~/git/K8s-13/code$ kubectl apply -f backend.yaml 
deployment.apps/backend created
service/backend created
serg@k8snode:~/git/K8s-13/code$ kubectl apply -f cache.yaml 
deployment.apps/cache created
service/cache created
serg@k8snode:~/git/K8s-13/code$ kubectl get -n app deployments
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
backend    1/1     1            1           25s
cache      1/1     1            1           17s
frontend   1/1     1            1           33s
serg@k8snode:~/git/K8s-13/code$ kubectl config set-context --current --namespace=app
Context "microk8s" modified.
serg@k8snode:~/git/K8s-13/code$ kubectl get pod -o wide
NAME                        READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
backend-65d8d9d74-swr4f     1/1     Running   0          62s   10.1.218.156   k8sclaster   <none>           <none>
cache-bfdc876fb-zlqf6       1/1     Running   0          54s   10.1.218.146   k8sclaster   <none>           <none>
frontend-59dcd68694-gqf6k   1/1     Running   0          70s   10.1.218.137   k8sclaster   <none>           <none>
serg@k8snode:~/git/K8s-13/code$ kubectl get svc -o wide
NAME       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE   SELECTOR
backend    ClusterIP   10.152.183.141   <none>        80/TCP    77s   app=backend
cache      ClusterIP   10.152.183.163   <none>        80/TCP    69s   app=cache
frontend   ClusterIP   10.152.183.126   <none>        80/TCP    85s   app=frontend

```

Проверяем доступность сервисов:

```
serg@k8snode:~/git/K8s-13/code$ kubectl exec frontend-59dcd68694-gqf6k -- curl backend
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    79  100    79    0     0  49716      0 --:--:-- --:--:-- --:--:-- 79000
Praqma Network MultiTool (with NGINX) - backend-65d8d9d74-swr4f - 10.1.218.156
serg@k8snode:~/git/K8s-13/code$ kubectl exec cache-bfdc876fb-zlqf6 -- curl backend
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    79  100    79    0     0  94837      0 --:-Praqma Network MultiTool (with NGINX) - backend-65d8d9d74-swr4f - 10.1.218.156
-:-- --:--:-- --:--:-- 79000
serg@k8snode:~/git/K8s-13/code$ kubectl exec backend-65d8d9d74-swr4f -- curl backend
Praqma Network MultiTool (with NGINX) - backend-65d8d9d74-swr4f - 10.1.218.156
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    79  100    79    0     0  62401      0 --:--:-- --:--:-- --:--:-- 79000
serg@k8snode:~/git/K8s-13/code$ kubectl exec cache-bfdc876fb-zlqf6 -- curl frontend
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    81  100    81    0     0  16068      0 --:--:-- --:--:-- --:--:-- 20250
Praqma Network MultiTool (with NGINX) - frontend-59dcd68694-gqf6k - 10.1.218.137
serg@k8snode:~/git/K8s-13/code$ kubectl exec backend-65d8d9d74-swr4f -- curl cache
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    77  100    77    0     0  57037      0 --:--:-- --:--:-- --:--:-- 77000
Praqma Network MultiTool (with NGINX) - cache-bfdc876fb-zlqf6 - 10.1.218.146

```

Создаем сетевые политики:

[default.yaml](https://github.com/smabramov/K8s-13/blob/413dfab4a668de15f3b6f1046f2ba19933b756bf/code/policy/default.yaml)

[policy_backend.yaml](https://github.com/smabramov/K8s-13/blob/413dfab4a668de15f3b6f1046f2ba19933b756bf/code/policy/policy_backend.yaml)

[policy_cache.yaml](https://github.com/smabramov/K8s-13/blob/413dfab4a668de15f3b6f1046f2ba19933b756bf/code/policy/policy_cache.yaml)

```

Создаем общее запрещающее правило и сразу проверяем:

```
serg@k8snode:~/git/K8s-13/code/policy$ kubectl apply -f default.yaml
networkpolicy.networking.k8s.io/default-deny-ingress created
serg@k8snode:~/git/K8s-13/code/policy$ kubectl exec frontend-59dcd68694-gqf6k -- curl cache
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:02:15 --:--:--     0
curl: (28) Failed to connect to cache port 80 after 135285 ms: Operation timed out
command terminated with exit code 28
```

Разрешающие правила для наших подов:

```
serg@k8snode:~/git/K8s-13/code/policy$ kubectl apply -f policy_cache.yaml 
networkpolicy.networking.k8s.io/cache created
serg@k8snode:~/git/K8s-13/code/policy$ kubectl apply -f policy_backend.yaml 
networkpolicy.networking.k8s.io/backend created
serg@k8snode:~/git/K8s-13/code/policy$ kubectl get networkpolicy
NAME                   POD-SELECTOR   AGE
backend                app=backend    15s
cache                  app=cache      28s
default-deny-ingress   <none>         6m48s
```

Проверяем:

```
serg@k8snode:~/git/K8s-13/code/policy$ kubectl exec frontend-59dcd68694-gqf6k -- curl --max-time 10 backend
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    79  100    79    0     0  98873      0 --:--:-- --:--:-- --:--:-- 79000
Praqma Network MultiTool (with NGINX) - backend-65d8d9d74-swr4f - 10.1.218.156
serg@k8snode:~/git/K8s-13/code/policy$ kubectl exec frontend-59dcd68694-gqf6k -- curl --max-time 10 cache
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:10 --:--:--     0
curl: (28) Connection timed out after 10000 milliseconds
command terminated with exit code 28
serg@k8snode:~/git/K8s-13/code/policy$ kubectl exec backend-65d8d9d74-swr4f -- curl --max-time 10 cache
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    77  100    77    0     0  64651      0 --:--:-- --:--:-- --:--:-- 77000
Praqma Network MultiTool (with NGINX) - cache-bfdc876fb-zlqf6 - 10.1.218.146
serg@k8snode:~/git/K8s-13/code/policy$ kubectl exec backend-65d8d9d74-swr4f -- curl --max-time 10 frontend
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:10 --:--:--     0
curl: (28) Connection timed out after 10000 milliseconds
command terminated with exit code 28
serg@k8snode:~/git/K8s-13/code/policy$ kubectl exec cache-bfdc876fb-zlqf6 -- curl --max-time 10 frontend
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:10 --:--:--     0
curl: (28) Connection timed out after 10000 milliseconds
command terminated with exit code 28
serg@k8snode:~/git/K8s-13/code/policy$ kubectl exec cache-bfdc876fb-zlqf6 -- curl --max-time 10 backend
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:09 --:--:--     0
curl: (28) Connection timed out after 10000 milliseconds
command terminated with exit code 28

```




### Правила приёма работы

1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.
