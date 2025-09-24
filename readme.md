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
| `nameOverride` | Переопределение имени чарта | `""` |
| `namespaceOverride` | Переопределение namespace | `""` |
| `fullnameOverride` | Полное переопределение имени | `""` |
| `global.imagePullSecrets` | Секреты для pull образов | `[docker-registry]` |
| `secretName` | Имя секрета с учетными данными SberCloud | `csi-evs-controller` |
| **Controller параметры** | | |
| `controller.serviceAccount` | Service Account для контроллера | `csi-evs-controller-sa` |
| `controller.replicas` | Количество реплик контроллера | `1` |
| `controller.nodeSelector` | Селектор узлов для контроллера | `kubernetes.io/os: linux` |
| `controller.priorityClassName` | Класс приоритета для контроллера | `system-cluster-critical` |
| `controller.tolerations` | Tolerations для контроллера | `[node-role.kubernetes.io/master: NoSchedule]` |
| `controller.imagePullSecrets` | Секреты для pull образов контроллера | `[docker-registry]` |
| `controller.labels` | Лейблы для контроллера | `app: csi-evs-provisioner` |
| **Controller образы** | | |
| `controller.csiAttacher.image.name` | CSI Attacher образ | `k8s.gcr.io/sig-storage/csi-attacher` |
| `controller.csiAttacher.image.tag` | CSI Attacher тег | `v3.3.0` |
| `controller.csiAttacher.image.imagePullPolicy` | CSI Attacher pull policy | `IfNotPresent` |
| `controller.csiProvisioner.image.name` | CSI Provisioner образ | `k8s.gcr.io/sig-storage/csi-provisioner` |
| `controller.csiProvisioner.image.tag` | CSI Provisioner тег | `v3.0.0` |
| `controller.csiProvisioner.image.imagePullPolicy` | CSI Provisioner pull policy | `IfNotPresent` |
| `controller.csiSnapshotter.image.name` | CSI Snapshotter образ | `k8s.gcr.io/sig-storage/csi-snapshotter` |
| `controller.csiSnapshotter.image.tag` | CSI Snapshotter тег | `v4.2.1` |
| `controller.csiSnapshotter.image.imagePullPolicy` | CSI Snapshotter pull policy | `Always` |
| `controller.csiResizer.image.name` | CSI Resizer образ | `k8s.gcr.io/sig-storage/csi-resizer` |
| `controller.csiResizer.image.tag` | CSI Resizer тег | `v1.3.0` |
| `controller.csiResizer.image.imagePullPolicy` | CSI Resizer pull policy | `IfNotPresent` |
| `controller.evsCsiProvisioner.image.name` | EVS CSI Provisioner образ | `swr.cn-north-4.myhuaweicloud.com/k8s-csi/evs-csi-plugin` |
| `controller.evsCsiProvisioner.image.tag` | EVS CSI Provisioner тег | `v0.1.11` |
| `controller.evsCsiProvisioner.image.imagePullPolicy` | EVS CSI Provisioner pull policy | `IfNotPresent` |
| `controller.livenessProbe.image.name` | Liveness Probe образ | `k8s.gcr.io/sig-storage/livenessprobe` |
| `controller.livenessProbe.image.tag` | Liveness Probe тег | `v2.5.0` |
| `controller.livenessProbe.image.imagePullPolicy` | Liveness Probe pull policy | `IfNotPresent` |
| **Node Plugin параметры** | | |
| `nodePlugin.serviceAccount` | Service Account для node plugin | `csi-evs-node-sa` |
| `nodePlugin.nodeSelector` | Селектор узлов для node plugin | `kubernetes.io/os: linux` |
| `nodePlugin.tolerations` | Tolerations для node plugin | `[operator: Exists]` |
| `nodePlugin.priorityClassName` | Класс приоритета для node plugin | `""` |
| `nodePlugin.labels` | Лейблы для node plugin | `app: csi-evs-plugin` |
| **Node Plugin образы** | | |
| `nodePlugin.evsDriverRegistrar.image.name` | Driver Registrar образ | `k8s.gcr.io/sig-storage/csi-node-driver-registrar` |
| `nodePlugin.evsDriverRegistrar.image.tag` | Driver Registrar тег | `v2.4.0` |
| `nodePlugin.evsDriverRegistrar.image.imagePullPolicy` | Driver Registrar pull policy | `IfNotPresent` |
| `nodePlugin.evsCsiPlugin.image.name` | EVS CSI Plugin образ | `swr.cn-north-4.myhuaweicloud.com/k8s-csi/evs-csi-plugin` |
| `nodePlugin.evsCsiPlugin.image.tag` | EVS CSI Plugin тег | `v0.1.11` |
| `nodePlugin.evsCsiPlugin.image.imagePullPolicy` | EVS CSI Plugin pull policy | `IfNotPresent` |
| `nodePlugin.livenessProbe.image.name` | Liveness Probe образ | `k8s.gcr.io/sig-storage/livenessprobe` |
| `nodePlugin.livenessProbe.image.tag` | Liveness Probe тег | `v2.5.0` |
| `nodePlugin.livenessProbe.image.imagePullPolicy` | Liveness Probe pull policy | `IfNotPresent` |
| **CSIDriver параметры** | | |
| `CSIDriver.attachRequired` | Требуется ли attach для драйвера | `true` |
| `CSIDriver.podInfoOnMount` | Передавать информацию о поде при mount | `true` |
| `CSIDriver.volumeLifecycleModes` | Режимы жизненного цикла томов | `[Persistent, Ephemeral]` |

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

