# Quay Registry on OpenShift with MinIO for Multi-Cluster

## 1.MinIO on OpenShift

### 1-1.Prepare PV

Prepare persistent volume without the Local Storage Operator after attaching virtual volume to VM.
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: minio-local-storage-pv
spec:
  capacity:
    storage: 190Gi
  volumeMode: Filesystem 
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: local-storage
  local:
    path: /dev/vdb
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - ocp4-hub.edwin.home
```

- https://docs.openshift.com/container-platform/4.14/storage/persistent_storage/persistent_storage_local/persistent-storage-local.html#local-create-cr-manual_persistent-storage-local - `OpenShift Local Storage`

### 1-2.Deploy MinIO

Deploy an instance development purpose
- https://min.io/docs/minio/kubernetes/openshift/index.html - `MinIO Dev Mode`

### 1-3. Create bucket for Quay

## 2. Install Quay

### 2-1. Pre configuration

create `config.yaml`` file
```yaml
FEATURE_USER_INITIALIZE: true
BROWSER_API_CALLS_XHR_ONLY: false
SUPER_USERS:
  - quayadmin
FEATURE_USER_CREATION: false
DISTRIBUTED_STORAGE_CONFIG:
  minioStorage:
    - RadosGWStorage
    - hostname: minio.minio.svc.cluster.local
      access_key: xxx
      secret_key: xxx
      bucket_name: quay
      is_secure: false
      port: 9000
      storage_path: /datastorage/registry
DISTRIBUTED_STORAGE_DEFAULT_LOCATIONS: []
DISTRIBUTED_STORAGE_PREFERENCE:
    - minioStorage
FEATURE_PROXY_CACHE: true
FEATURE_QUOTA_MANAGEMENT: true
# Enable 3.10 features
FEATURE_UI_V2: true
FEATURE_UI_V2_REPO_SETTINGS: true
FEATURE_AUTO_PRUNE: true
ROBOTS_DISALLOW: false
```
```bash
oc create secret -n quay-enterprise generic --from-file config.yaml=./config.yaml init-config-bundle-secret
```

### 2-2. Create a Quay instance

- quay-registry definition
```yaml
kind: QuayRegistry
apiVersion: quay.redhat.com/v1
metadata:
  name: quay-registry
spec:
  configBundleSecret: init-config-bundle-secret
  components:
    - kind: clair
      managed: false
    - kind: clairpostgres
      managed: false
    - kind: postgres
      managed: true
      overrides:
        resources: ''
    - kind: objectstorage
      managed: false
    - kind: redis
      managed: true
    - kind: route
      managed: true
    - kind: mirror
      managed: false
    - kind: monitoring
      managed: false
    - kind: quay
      managed: true
      overrides:
        replicas: 1
    - kind: horizontalpodautoscaler
      managed: false
```

```bash
oc create -n quay-enterprise -f quayregistry.yaml
```

#Quay bridge operator configuration

```
oc create secret -n openshift-operators generic quay-edwin --from-literal=token=d0P4kDYT2tKhQ3xaki422HgaUtMmYGbExtn04snh
```
