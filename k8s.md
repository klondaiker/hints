# K8s
## Команды K8s
- `kubectl version` - показать версию клиента и сервера
- `kubectl config get-contexts` - показать список контекстов
- `kubectl config use-context $CONTEXT` - использовать контекст
- `kubectl get componentstatuses` - показать состояние кластера
- `kubectl cluster-info` - показать инфо о кластере
- `kubectl get nodes` - показать все серверы кластера
- `kubectl get ns` - показать список пространств
- `kubectl get pods -n $NAMESPACE` - показать все поды пространства
- `kubectl exec -itn $NAMESPACE $POD -- $COMMAND` - запустить команду внутри пода
- `kubectl logs -f -n $NAMESPACE  $POD` - логи пода
- `kubectl get deployments -n $NAMESPACE` - показать все деплоименты пространства
- `kubectl logs -f deployment/$DEPLOYMENT -n $NAMESPACE` - логи деплоймента
- `kubectl scale --replicas=$N deployment/$DEPLOYMENT -n $NAMESPACE` - настройки реплики деплоймента

## Устройство k8s
Главные объекты k8s:
- Cluster: Логическое объединение Nodes
- Node: Сервера, где все это работает. Может быть Master Node(kube-apiserver, kuber-controller-manager, kuber-scheduler) или Worker Node(kubelet, kuber-proxy)
- Service: предоставляет доступ к Deployment через ClusterIP, NodePort, LoadBalancer, ExternalName
- Deployment: Cэт одинаковых Pods, нужен для autoscaling и для обновления Container Image, держит минимальное количество работающих Pod
- Pod: Объект, в котором работают один или больше Contatiner
- Contatiner: Docker контейнера

Другие объекты k8s:
DeamonSets, StatefulSets, ReplicaSet, Secrets, PV, SVC, LoadBalancers, ConfigMaps, Vertical Pod Autoscaler, HorizontalPodAutoscaler
