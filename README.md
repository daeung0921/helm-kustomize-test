
```
$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm repo update bitnami
$  helm search repo bitnami/postgresql --versions    
NAME                    CHART VERSION   APP VERSION     DESCRIPTION
bitnami/postgresql      13.2.24         16.1.0          PostgreSQL (Postgres) is an open source object-...
bitnami/postgresql      13.2.23         16.1.0          PostgreSQL (Postgres) is an open source object-...
bitnami/postgresql      13.2.22         16.1.0          PostgreSQL (Postgres) is an open source object-...
bitnami/postgresql      13.2.21         16.1.0          PostgreSQL (Postgres) is an open source object-...
...

# values 받기
$ helm inspect values bitnami/postgresql --version  13.2.24   > my-values.yaml

# helm template manifest 확인
$ helm template my-postgre bitnami/postgresql -f my-values.yaml 
 
# 신규 리소스
$ sudo tee namespace.yaml > /dev/null <<EOF
apiVersion: v1
kind: Namespace
metadata:
  name: test
EOF

# kustomization 
$ sudo tee kustomization.yaml > /dev/null <<EOF
resources:
- namespace.yaml

helmChartInflationGenerator:
- chartName: postgresql
  chartRepoUrl: https://charts.bitnami.com/bitnami
  chartVersion: 13.2.24
  releaseName: my-postgre-release
  releaseNamespace: postgre-chart 
  values: my-values.yaml
EOF

# kustomize manifest 확인
$ kubectl kustomize --enable-helm
apiVersion: v1
kind: Namespace
metadata:
  name: test
---
apiVersion: v1
data:
  postgres-password: a01aRkdTaXFUZA==
kind: Secret
metadata:
  labels:
    app.kubernetes.io/instance: my-postgre-release
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: postgresql
    app.kubernetes.io/version: 16.1.0
    helm.sh/chart: postgresql-13.2.24
  name: my-postgre-release-postgresql
  namespace: default
type: Opaque
---
apiVersion: v1
kind: Service
metadata:
...

# 배포
$ kubectl kustomize --enable-helm | kubectl apply -f -
```
