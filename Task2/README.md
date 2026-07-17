# Задание 2. Динамическое масштабирование контейнеров

## Часть 1. Динамическая маршрутизация на основании показателей утилизации памяти

### 1. Подготовка окружения

```bash
cd ./Task2

minikube start --driver=docker
minikube addons enable metrics-server
```

### 2. Манифест развертывания

Применяем манифест [deployment.yaml](deployment.yaml)
```bash
kubectl apply -f deployment.yaml
```

### 3. Манифест сервиса

Применяем манифест [service.yaml](service.yaml)
```bash
kubectl apply -f service.yaml
```

### 4. Настройка Horizontal Pod Autoscaler (HPA)

Применяем манифест [hpa.yaml](hpa.yaml)
```bash
kubectl apply -f hpa.yaml
```

Проверка
```bash
kubectl get hpa
```

### 5. Запуск нагрузочного теста

Устанавливаем locust и запускаем
```bash
brew install locust
locust
```
**Number of users:** 300 \
**Rump up**: 10

![hpa_watch.png](image/hpa_watch.png)

![dashboard_1.png](image/dashboard_1.png)

![dashboard_2.png](image/dashboard_2.png)

![dashboard_3.png](image/dashboard_3.png)

Логи [hpa-events.txt](hpa-events.txt)
```
Name:                                                     scaletestapp-hpa
Namespace:                                                default
Labels:                                                   <none>
Annotations:                                              <none>
CreationTimestamp:                                        Fri, 17 Jul 2026 10:01:35 +0300
Reference:                                                Deployment/scaletestapp
Metrics:                                                  ( current / target )
  resource memory on pods  (as a percentage of request):  81% (25096Ki) / 80%
Min replicas:                                             1
Max replicas:                                             10
Deployment pods:                                          2 current / 2 desired
Conditions:
  Type            Status  Reason              Message
  ----            ------  ------              -------
  AbleToScale     True    ReadyForNewScale    recommended size matches current size
  ScalingActive   True    ValidMetricFound    the HPA was able to successfully calculate a replica count from memory resource utilization (percentage of request)
  ScalingLimited  False   DesiredWithinRange  the desired count is within the acceptable range
Events:
  Type    Reason             Age   From                       Message
  ----    ------             ----  ----                       -------
  Normal  SuccessfulRescale  10m   horizontal-pod-autoscaler  New size: 2; reason: memory resource utilization (percentage of request) above target
```

### 6. Анализ результатов масштабирования

В ходе нагрузочного тестирования HPA успешно отработал в соответствии с заданными параметрами:
- **Целевой уровень утилизации памяти:** 80% от запрошенного лимита (`30 Mi`).
- **Фактическое потребление памяти:** в момент срабатывания HPA составило 81% (`25096 KiB`), что превысило целевой порог.
- **Реакция HPA:** количество реплик было увеличено с 1 до 2, о чем свидетельствует событие `SuccessfulRescale` в логах.
- **Состояние HPA:** все условия (`AbleToScale`, `ScalingActive`, `ScalingLimited`) имеют статус True или False с корректными причинами, что подтверждает корректную работу механизма.

Таким образом, динамическое масштабирование на основе утилизации памяти функционирует правильно и позволяет системе автоматически адаптироваться к росту нагрузки.

