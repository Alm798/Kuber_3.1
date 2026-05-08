Домашнее задание к занятию «Компоненты Kubernetes»
Задача 1
Необходимо определить требуемые ресурсы для Kubernetes-кластера и подобрать оптимальное количество worker-нод для размещения приложения.

Исходные данные
Приложение состоит из:

Компонент	Реплики	CPU	RAM
База данных	3	1 CPU	4 GB
Кеш	3	1 CPU	4 GB
Backend	10	1 CPU	600 MB
Frontend	5	0.2 CPU	50 MB
Также приложение должно поддерживать deployment через Helm chart в окружения:

dev
stage
prod
Решение
Расчёт ресурсов
База данных
3 × 1 CPU = 3 CPU
3 × 4 GB = 12 GB RAM
Кеш
3 × 1 CPU = 3 CPU
3 × 4 GB = 12 GB RAM
Backend
10 × 1 CPU = 10 CPU
10 × 600 MB = 6000 MB ≈ 6 GB RAM
Frontend
5 × 0.2 CPU = 1 CPU
5 × 50 MB = 250 MB ≈ 0.25 GB RAM
Общий объём ресурсов
Ресурс	Значение
CPU	17
RAM	~30.25 GB
Дополнительные условия
Помимо самих контейнеров необходимо учитывать:

kubelet
kube-proxy
container runtime
CNI plugin
мониторинг
логирование
системные процессы ОС
Также кластер должен сохранять работоспособность при отказе одной worker-ноды.

Архитектура кластера
Для более корректного распределения нагрузки разделяю worker-ноды на два пула:

Stateful pool
Для:

БД
кеша
Используются:

StatefulSet
PVC
podAntiAffinity
Конфигурация
3 worker-ноды
4 CPU
16 GB RAM
Общие ресурсы
12 CPU
48 GB RAM
Stateless pool
Для:

backend
frontend
Используются:

Deployment
HPA
Ingress
Конфигурация
3 worker-ноды
8 CPU
16 GB RAM
Общие ресурсы
24 CPU
48 GB RAM
Даже при потере одной ноды останется:

16 CPU
32 GB RAM
что достаточно для работы frontend/backend и служебных компонентов Kubernetes.

Итоговая конфигурация
Пул	Ноды	CPU	RAM
Stateful	3	4 CPU	16 GB
Stateless	3	8 CPU	16 GB
Итог
Оптимальная конфигурация Kubernetes-кластера:

6 worker-нод
36 CPU
96 GB RAM
Данная конфигурация обеспечивает:

отказоустойчивость
запас ресурсов
масштабируемость
разделение stateful/stateless workload
удобный deployment через Helm chart
возможность дальнейшего горизонтального масштабирования
Используемые Kubernetes-объекты
Stateful
StatefulSet
PersistentVolumeClaim
StorageClass
PodDisruptionBudget
podAntiAffinity
Stateless
Deployment
Service
Ingress
HorizontalPodAutoscaler
readinessProbe
livenessProbe
Helm
values-dev.yaml
values-stage.yaml
values-prod.yaml
