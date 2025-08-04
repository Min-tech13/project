# Проект Автоматизации Инфраструктуры

Данный проект демонстрирует полную настройку инфраструктуры с использованием Ansible для автоматизации, Kubernetes для оркестрации контейнеров и различных решений мониторинга.

## Структура Проекта

```
├── ansible/                    # Скрипты автоматизации Ansible
│   ├── roles/
│   │   ├── docker/            # Роль установки Docker
│   │   ├── kube/              # Роль установки Kubernetes
│   │   ├── kafka/             # Настройка кластера Kafka
│   │   └── postgres/          # Настройка базы данных PostgreSQL
├── docker/                    # Конфигурации Docker
├── kubernetes/                # Манифесты Kubernetes и Helm чарты
│   └── simple-nginx/          # Helm чарт Nginx
├── monitoring/                # Команды настройки мониторинга
└── kubeadm-config.yml        # Конфигурация кластера Kubernetes
```

## Предварительные Требования

- Несколько Linux серверов (1 control plane + 2 worker узла)
- Ansible установлен на управляющей машине
- SSH доступ ко всем целевым серверам

## Инструкции по Настройке

### 1. Подготовка Инфраструктуры с Помощью Ansible

Сначала настройте все узлы с Docker и предварительными требованиями Kubernetes:

```bash
# Запустите Ansible playbook для установки Docker и Kubernetes на всех узлах
ansible-playbook -i inventory.ini playbook.yml
```

Этот playbook включает:
- **роль docker**: Устанавливает Docker на всех узлах
- **роль kube**: Устанавливает компоненты Kubernetes (kubelet, kubeadm, kubectl)

### 2. Инициализация Кластера Kubernetes

#### Инициализация Control Plane

На узле control plane:

```bash
# Инициализация кластера с пользовательской конфигурацией
sudo kubeadm init --config kubeadm-config.yml
```

#### Настройка Доступа kubectl

```bash
# Настройка kubectl для текущего пользователя
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### Проверка Первоначальной Настройки

```bash
# Проверка статуса узла (изначально должен показывать NotReady)
kubectl get nodes
```

#### Установка Сетевого Дополнения Calico

```bash
# Установка плагина Calico CNI
kubectl apply -f https://docs.projectcalico.org/v3.14/manifests/calico.yaml

# Проверка что узел теперь Ready
kubectl get nodes
```

#### Присоединение Worker Узлов

Получите команду присоединения с control plane:

```bash
# Генерация команды присоединения
kubeadm token create --print-join-command
```

Выполните команду присоединения на каждом worker узле:

```bash
# На worker узлах (выполнить как root)
sudo kubeadm join <control-plane-ip>:6443 --token <token> --discovery-token-ca-cert-hash <hash>
```

#### Проверка Кластера

```bash
# Подтверждение что все узлы Ready
kubectl get nodes
```

### 3. Настройка Базы Данных и Системы Сообщений

Развертывание кластеров PostgreSQL и Kafka с помощью Ansible:

```bash
# Развертывание кластера PostgreSQL с репликацией
ansible-playbook -i inventory.ini playbook.yml --tags postgres

# Развертывание кластера Kafka
ansible-playbook -i inventory.ini playbook.yml --tags kafka
```

**Возможности PostgreSQL:**
- Настройка репликации master-slave
- Пользовательские шаблоны конфигурации
- Автоматизированное управление сервисами

**Возможности Kafka:**
- Мульти-брокерный кластер
- Пользовательские свойства сервера
- Управление сервисами и мониторинг

### 4. Docker Приложение

Настройка простого Docker приложения:

```bash
# Переход в директорию docker
cd docker/

# Сборка Docker образа
docker build -t simple-app .

# Запуск контейнера
docker run -d -p 80:80 simple-app
```

### 5. Развертывание Kubernetes Приложения

Развертывание приложений с использованием Helm чартов:

```bash
# Переход в директорию kubernetes
cd kubernetes/

# Установка Nginx приложения с помощью Helm
helm install simple-nginx ./simple-nginx/

# Проверка развертывания
kubectl get pods
kubectl get services
```

**Возможности:**
- Helm чарт для легкого развертывания
- Настроенный Horizontal Pod Autoscaler (HPA)
- Ingress контроллер для внешнего доступа
- Готовность к service mesh

#### Horizontal Pod Autoscaler

HPA настроен для автоматического масштабирования подов на основе использования CPU:

```bash
# Проверка статуса HPA
kubectl get hpa

# Применение конфигурации HPA
kubectl apply -f hpa.yaml
```

### 6. Настройка Мониторинга

Установка Prometheus для мониторинга кластера:

```bash
# Переход в директорию monitoring
cd monitoring/

# Выполнение команд настройки мониторинга
./commands
```

Это включает:
- Установка сервера Prometheus
- Конфигурация сбора метрик
- Настройка дашборда для мониторинга кластера

## Обзор Архитектуры

1. **Уровень Инфраструктуры**: Автоматизирован с помощью Ansible
   - Docker runtime на всех узлах
   - Настройка кластера Kubernetes
   - Сервисы баз данных и системы сообщений

2. **Оркестрация Контейнеров**: Kubernetes
   - Мульти-узловой кластер с сетью Calico
   - Helm чарты для развертывания приложений
   - Возможности автомасштабирования

3. **Уровень Данных**: 
   - PostgreSQL с репликацией
   - Kafka для потоковой передачи сообщений

4. **Мониторинг**: Наблюдаемость на основе Prometheus

## Ключевые Компоненты

- **Роли Ansible**: Модульная автоматизация для настройки инфраструктуры
- **Кластер Kubernetes**: Готовая к продакшену оркестрация контейнеров
- **Helm Чарты**: Шаблонизированные развертывания Kubernetes
- **Автомасштабирование**: HPA для динамического управления ресурсами
- **Мониторинг**: Комплексный сбор метрик

## Использование

1. Запустите Ansible playbooks для подготовки инфраструктуры
2. Инициализируйте кластер Kubernetes следуя пошаговому руководству
3. Разверните приложения используя предоставленные Helm чарты
4. Мониторьте систему используя дашборды Prometheus

## Примечания

- Убедитесь что все узлы соответствуют системным требованиям Kubernetes
- Требуется сетевое подключение между всеми узлами
- Данные мониторинга сохраняются согласно политикам хранения Prometheus
- HPA требует запущенного metrics-server в кластере