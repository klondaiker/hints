# K8s
## Устройство k8s
Главные объекты k8s:
- Container: Docker-контейнер
- Pod: Объект, в котором работают один или больше Contatiner
- Deployment: Сэт одинаковых Pods, нужен для autoscaling и для обновления Container Image, держит минимальное количество работающих Pod
- Service: предоставляет доступ к Deployment через следующие типы:
  1) ClusterIP: Обеспечивает сервис внутри кластера, к которому могут обращаться другие приложения внутри кластера. Внешнего доступа нет. 
  2) NodePort: Открывает указанный порт для всех Nodes (виртуальных машин), и трафик на этот порт перенаправляется сервису. Метод имеет множество недостатков: На порт садится только один сервис и доступны только порты 30000–32767
  3) LoadBalancer:  На GKE он развернет Network Load Balancer, который предоставит IP адрес. Этот IP адрес будет направлять весь трафик на сервис. Но есть один недостаток. Каждому сервису, который мы раскрываем с помощью LoadBalancer, нужен свой IP-адрес, что может влететь в копеечку
  4) ExternalName: Сопоставляет службу с содержимым поля externalName (например, с именем хоста api.foo.bar.example). 
- Ingress: Занимается маршрутизацией к сервисам. Самым популярным является nginx-ingress
- Node: Сервера, где все это работает. Может быть Master Node(kube-apiserver, kuber-controller-manager, kuber-scheduler) или Worker Node(kubelet, kuber-proxy)
- Cluster: Логическое объединение Nodes

Другие объекты k8s: \
DeamonSets, StatefulSets, ReplicaSet, Secrets, PV, SVC, LoadBalancers, ConfigMaps, Vertical Pod Autoscaler, HorizontalPodAutoscaler

## Команды K8s
- `kubectl version` - показать версию клиента и сервера

Context:
- `kubectl config get-contexts` - показать список контекстов
- `kubectl config use-context $CONTEXT` - использовать контекст

Cluster: 
- `kubectl get componentstatuses` - показать состояние кластера
- `kubectl cluster-info` - показать инфо о кластере

Nodes:
- `kubectl get nodes` - показать все серверы кластера

Namespaces:
- `kubectl get ns` - показать список пространств

Service:
- `kubectl get service` - показать список сервисов
- `kubectl expose deployment $DEPLOYMENT --type=$TYPE --port $PORT` - создает сервис к Deployment
- `kubectl delete service $DEPLOYMENT` - удаление сервиса

Deployments:
- `kubectl get deployments -n $NAMESPACE` - показать все деплоименты пространства
- `kubectl logs -f deployment/$DEPLOYMENT -n $NAMESPACE` - логи деплоймента
- `kubectl create deployment $DEPLOYMENT --image $IMAGE` - создание деплоймента
- `kubectl delete deployment $DEPLOYMENT` - удаление деплоймента
- `kubectl describe deployment $DEPLOYMENT` - информация о деплойменте
- `kubectl scale deployment $DEPLOYMENT --replicas $N -n $NAMESPACE` - скалировать реплики деплоймента
- `kubectl autoscale deployment $DEPLOYMENT --min=4 --max=6 --cpu-percent=80` - автоскалирование деплоймента (создание HorizontalPodAutoscaler)
- `kubectl rollout history deployment $DEPLOYMENT` - история деплоймента
- `kubectl rollout status deployment $DEPLOYMENT` - статус деплоймента
- `kubectl rollout undo deployment $DEPLOYMENT --to-revision=$N` - возврат на предыдующую версию

Pods:
- `kubectl get pods -n $NAMESPACE` - показать все поды пространства
- `kubectl exec -itn $NAMESPACE $POD -- $COMMAND` - запустить команду внутри пода
- `kubectl logs -f -n $NAMESPACE  $POD` - логи пода
- `kubectl run hello --image=$IMAGE --port=$PORT` - создание пода
- `kubectl delete pod hello` - удаление пода
- `kubectl describe pods %POD` - информация о поде
- `kubectl port-forward %POD $PORT:$PORT` - перенаправление порта

Config:
- `kubectl apply -f example.yaml` - примененение манифест-файла
- `kubectl delete -f example.yaml` - удаление манифеста

## Манифесты
Позволяет описывать команды в yaml-файлах
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: standalone-pod
spec:
  containers:
  - name: main
    image: alpine
    command: ["tail", "-f", "/dev/null"]
```
## Helm
Разрешает проблему дубликатов манифестов при их изменении
<img width="1454" alt="Снимок экрана 2025-02-14 в 02 22 01" src="https://github.com/user-attachments/assets/e0bd71c9-a0cf-4c87-bd6e-39a0cfa2a457" />

## Команды Helm
- `helm version` - показать версию helm
- `helm list` - список всех деплоев
- `helm install $APP $CHART` - деплой чарта (может из папки или архива)
- `helm upgrade $APP $CHART_FOLDER/` - обновления деплоя
- `helm package $CHART_FOLDER/` - запаковать чарт в архив
- `helm delete $APP` - удалить деплой


## Volume
