## 🧠 Основы Kubernetes

**Kubernetes** — это система оркестрации контейнеров с открытым исходным кодом для автоматизации развертывания, масштабирования и управления приложениями в контейнерах.

### Основные компоненты Kubernetes:
- **Cluster**: Совокупность рабочих машин, на которых работают приложения. Каждый кластер состоит из двух типов узлов (nodes): 
  - **Master** — управляет кластером, отвечает за планирование задач, управление состоянием и т.д.
  - **Node** — физическая или виртуальная машина, на которой работают контейнеры.

---

## 🧩 Ключевые объекты в Kubernetes

1. **Pod**: Основная единица развертывания и масштабирования в Kubernetes. Контейнеры внутри Pod'а делят сеть и хранилище.
   - Например, если вам нужно, чтобы два контейнера (например, веб-сервер и база данных) работали вместе и использовали общий ресурс (том), то они могут быть развернуты в одном Pod'е.

2. **Service**: Абстракция, которая позволяет проксировать запросы к набору Pod'ов. Есть несколько типов:
   - **ClusterIP** (по умолчанию) — доступность внутри кластера.
   - **NodePort** — доступность через порты на всех узлах.
   - **LoadBalancer** — используется для автоматического создания балансировщика нагрузки в облаке.
   - **Headless Service** — нет собственного IP-адреса, используется для прямого доступа к Pod'ам.

3. **Deployment**: Управляет развертыванием и масштабированием наборов Pod'ов, позволяет легко откатывать или обновлять версии приложений.
   - Используется для контроля над количеством реплик Pod'ов.

4. **StatefulSet**: Похож на Deployment, но используется для приложений, которым нужно сохранять состояние. Предоставляет уникальные имена и стабильные хранилища для каждого экземпляра.

5. **ReplicaSet**: Управляет количеством реплик Pod'ов, работает совместно с Deployment для поддержания нужного количества экземпляров.

6. **ConfigMap** и **Secret**: Объекты для хранения конфигурационных данных, таких как переменные окружения и пароли. **ConfigMap** используется для общих данных, а **Secret** для более безопасного хранения (например, токенов и паролей).

7. **Volume**: Позволяет контейнерам хранить данные. Kubernetes поддерживает различные типы хранилищ, такие как:
   - **emptyDir** — временный диск, существующий только пока работает Pod.
   - **PersistentVolume** — внешний, более долговечный ресурс для хранения данных.

---

## 🔧 Основные команды `kubectl`

- **Получение информации**:
  ```bash
  kubectl get nodes        # Информация о нодах
  kubectl get pods         # Список подов
  kubectl get services     # Список сервисов
  kubectl get deployments  # Список деплойментов
  kubectl describe pod <pod_name>  # Подробности о pod-е
  ```

- **Создание объектов**:
  ```bash
  kubectl apply -f <file>.yaml    # Применяет конфигурацию из файла
  kubectl create -f <file>.yaml   # Создает объект из конфигурации
  ```

- **Удаление объектов**:
  ```bash
  kubectl delete -f <file>.yaml   # Удаляет объект, указанный в файле
  kubectl delete pod <pod_name>   # Удаляет конкретный pod
  ```

- **Отладка**:
  ```bash
  kubectl logs <pod_name>            # Логи конкретного pod-а
  kubectl exec -it <pod_name> -- bash # Доступ к контейнеру
  kubectl port-forward pod/<pod_name> 8080:80 # Прокси порта на локальной машине
  ```

- **Масштабирование приложений**:
  ```bash
  kubectl scale deployment <deployment_name> --replicas=3
  ```

---

## 🌍 Networking в Kubernetes

1. **CNI (Container Network Interface)**: Стандарт интерфейса для плагинов сети в Kubernetes. Некоторые популярные плагины:
   - **Calico**
   - **Weave**
   - **Flannel**

2. **Pod-to-Pod коммуникация**: Контейнеры в одном Pod'е могут общаться друг с другом через локальную сеть. Pod'ы могут общаться с другими Pod'ами в кластере с помощью их IP-адресов.

3. **Services**:
   - **ClusterIP**: По умолчанию, сервис доступен внутри кластера.
   - **NodePort**: Доступен снаружи кластера через порты всех узлов.
   - **LoadBalancer**: Для облачных провайдеров, автоматически создает балансировщик нагрузки.

---

## 🚀 Развертывание и управление приложениями

### Пример конфигурации для Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp-container
        image: myapp:v1
        ports:
        - containerPort: 8080
```

### Пример конфигурации для Service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: LoadBalancer
```

- **Deployment** управляет развертыванием, поддерживает нужное количество реплик.
- **Service** управляет сетевой доступностью.

---

## ⚙️ Ресурсы и управление ресурсами

1. **CPU и память**:
   - В Kubernetes можно ограничить количество ресурсов для контейнера.
   - Пример:
     ```yaml
     resources:
       requests:
         memory: "64Mi"
         cpu: "250m"
       limits:
         memory: "128Mi"
         cpu: "500m"
     ```

2. **Horizontal Pod Autoscaler (HPA)**:
   - Автоматическое масштабирование Pod'ов на основе использования ресурсов:
     ```bash
     kubectl autoscale deployment <deployment_name> --cpu-percent=50 --min=1 --max=10
     ```

3. **PodDisruptionBudget (PDB)**: Объект, который помогает управлять доступностью приложения при удалении Pod'ов:
   ```yaml
   apiVersion: policy/v1beta1
   kind: PodDisruptionBudget
   metadata:
     name: myapp-pdb
   spec:
     minAvailable: 1
     selector:
       matchLabels:
         app: myapp
   ```

---

## 🔄 Обновления и откаты

1. **Rolling Update**:
   Kubernetes поддерживает **rolling updates**, что позволяет обновлять приложение, не прекращая его работу:
   ```bash
   kubectl rollout restart deployment <deployment_name>
   ```

2. **Откат обновлений**:
   Если после обновления что-то пошло не так, можно откатить Deployment:
   ```bash
   kubectl rollout undo deployment <deployment_name>
   ```

---

## 🔐 Безопасность

1. **RBAC (Role-Based Access Control)**: Контролирует доступ на основе ролей.
   - **Role**: Определяет права в пределах одного namespace.
   - **ClusterRole**: Определяет глобальные права на уровне кластера.
   
2. **Network Policies**: Ограничивают сетевое взаимодействие между Pod'ами, например, разрешая доступ только для определенных Pod'ов.

3. **PodSecurityPolicy**: Политики безопасности для Pod'ов.

---

## 🌐 Подключение внешних сервисов

1. **Ingress**: Управляет внешним доступом к сервисам внутри кластера. Это более гибкое решение по сравнению с **LoadBalancer**.
   Пример конфигурации:
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: myapp-ingress
   spec:
     rules:
     - host: myapp.example.com
       http:
         paths:
         - path: /
           pathType: Prefix
           backend:
             service:
               name: myapp-service
               port:
                 number: 80
   ```
