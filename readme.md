# CSI EVS Controller Helm Chart

Helm чарт для развертывания CSI (Container Storage Interface) драйвера Huawei Cloud EVS (Elastic Volume Service) в Kubernetes кластерах SberCloud.

## Описание

Этот чарт предоставляет полнофункциональный CSI драйвер для работы с блочными хранилищами EVS в SberCloud. Драйвер поддерживает создание, подключение, отключение и удаление томов EVS, а также их расширение и создание снимков.

## Компоненты

- **CSI Controller**: Управляет жизненным циклом томов (создание, удаление, расширение, снимки)
- **CSI Node Plugin**: Управляет подключением/отключением томов на узлах кластера
- **RBAC**: Роли и привязки для безопасной работы с Kubernetes API
- **StorageClass**: Предопределенные классы хранилища для различных типов EVS

## Требования

- Kubernetes 1.19+
- Helm 3.0+
- SberCloud аккаунт с доступом к EVS API
- Настроенные секреты для аутентификации в SberCloud

## Установка

### 1. Подготовка секретов

Создайте секрет с учетными данными SberCloud:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: csi-evs-controller
  namespace: kube-system
type: Opaque
data:
  cloud-config: <base64-encoded-cloud-config>
```

Где `cloud-config` содержит:
```yaml
global:
  auth-url: "https://iam.ru-moscow-1.hc.sbercloud.ru/v3"
  username: "your-username"
  password: "your-password"
  domain-name: "your-domain"
  project-name: "your-project"
  region: "ru-moscow-1"
```

### 2. Установка чарта

```bash
helm install csi-evs-controller ./charts/csi-evs-controller \
  --namespace kube-system \
  --set secretName=csi-evs-controller
```

## Конфигурация

### Основные параметры

| Параметр | Описание | По умолчанию |
|----------|----------|--------------|
| `secretName` | Имя секрета с учетными данными SberCloud | `csi-evs-controller` |
| `controller.replicas` | Количество реплик контроллера | `1` |
| `controller.nodeSelector` | Селектор узлов для контроллера | `kubernetes.io/os: linux` |
| `nodePlugin.nodeSelector` | Селектор узлов для node plugin | `kubernetes.io/os: linux` |

### Образы контейнеров

Все образы полностью параметризованы и могут быть настроены в `values.yaml`:

#### Controller компоненты
- `controller.csiAttacher.image` - CSI Attacher
- `controller.csiProvisioner.image` - CSI Provisioner  
- `controller.csiResizer.image` - CSI Resizer
- `controller.csiSnapshotter.image` - CSI Snapshotter
- `controller.livenessProbe.image` - Liveness Probe
- `controller.evsCsiProvisioner.image` - EVS CSI Provisioner

#### Node Plugin компоненты
- `nodePlugin.evsDriverRegistrar.image` - Driver Registrar
- `nodePlugin.livenessProbe.image` - Liveness Probe
- `nodePlugin.evsCsiPlugin.image` - EVS CSI Plugin

### StorageClass

Чарт создает два класса хранилища по умолчанию:

#### EVS SAS
```yaml
storageClass:
  - name: evs
    allowVolumeExpansion: true
    parameters:
      type: SAS
      fsType: xfs
    reclaimPolicy: Delete
    volumeBindingMode: WaitForFirstConsumer
```

#### EVS SSD
```yaml
storageClass:
  - name: evs-ssd
    allowVolumeExpansion: true
    parameters:
      type: SSD
      fsType: xfs
    reclaimPolicy: Delete
    volumeBindingMode: WaitForFirstConsumer
```

### Настройка лейблов и селекторов

Чарт использует гибкую систему лейблов:

```yaml
controller:
  labels:
    app: csi-evs-provisioner
  # matchLabels генерируются автоматически

nodePlugin:
  labels:
    app: csi-evs-plugin
  # matchLabels генерируются автоматически
```

## Использование

### Создание PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: evs  # или evs-ssd
```

### Создание пода с томом

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
  - name: test-container
    image: busybox
    command: ['sleep', '3600']
    volumeMounts:
    - name: test-volume
      mountPath: /data
  volumes:
  - name: test-volume
    persistentVolumeClaim:
      claimName: test-pvc
```

## Мониторинг

### Проверка статуса

```bash
# Проверка подов контроллера
kubectl get pods -n kube-system -l app=csi-evs-provisioner

# Проверка подов node plugin
kubectl get pods -n kube-system -l app=csi-evs-plugin

# Проверка StorageClass
kubectl get storageclass

# Проверка CSIDriver
kubectl get csidriver evs.csi.huaweicloud.com
```

### Логи

```bash
# Логи контроллера
kubectl logs -n kube-system -l app=csi-evs-provisioner

# Логи node plugin
kubectl logs -n kube-system -l app=csi-evs-plugin
```

## Обновление

```bash
helm upgrade csi-evs-controller ./charts/csi-evs-controller \
  --namespace kube-system
```

## Удаление

```bash
# Удаление чарта
helm uninstall csi-evs-controller --namespace kube-system

# Удаление StorageClass (опционально)
kubectl delete storageclass evs evs-ssd
```

## Устранение неполадок

### Проблемы с аутентификацией
- Проверьте корректность учетных данных в секрете
- Убедитесь, что проект имеет права на работу с EVS

### Проблемы с подключением томов
- Проверьте, что node plugin запущен на всех узлах
- Убедитесь, что узел имеет доступ к SberCloud API

### Проблемы с созданием томов
- Проверьте логи контроллера
- Убедитесь, что StorageClass настроен корректно

## Поддерживаемые функции

- ✅ Создание и удаление томов EVS
- ✅ Подключение/отключение томов
- ✅ Расширение томов
- ✅ Создание снимков томов
- ✅ Поддержка различных типов EVS (SAS, SSD)
- ✅ WaitForFirstConsumer binding mode
- ✅ Параметризация всех образов
- ✅ Гибкая настройка лейблов и селекторов

## Версии

- **Chart Version**: 0.2.0
- **App Version**: 1.0.0
- **CSI Driver**: evs.csi.huaweicloud.com
- **EVS Plugin**: v0.1.11

## Лицензия

Этот чарт предназначен для использования в SberCloud и соответствует корпоративным стандартам безопасности.
