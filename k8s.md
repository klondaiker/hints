# K8s
## Устройство k8s
Главные объекты k8s:
- Cluster: Логическое объединение Nodes
- Node: Сервера, где все это работает. Может быть Master Node(kube-apiserver, kuber-controller-manager, kuber-scheduler) или Worker Node(kubelet, kuber-proxy)
- Service: предоставляет доступ к Deployment через ClusterIP, NodePort, LoadBalancer, ExternalName
- Deployment: Cэт одинаковых Pods, нужен для autoscaling и для обновления Container Image, держит минимальное количество работающих Pod
- Pod: Объект, в котором работают один или больше Contatiner
- Contatiner: Docker контейнера

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

Deployments:
- `kubectl get deployments -n $NAMESPACE` - показать все деплоименты пространства
- `kubectl logs -f deployment/$DEPLOYMENT -n $NAMESPACE` - логи деплоймента
- `kubectl create deployment $DEPLOYMENT --image $IMAGE` - создание деплоймента
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

