## Operator SDK - helm chart - gitea-charts/gitea 생성

- download operator-sdk

```
curl -LO https://github.com/operator-framework/operator-sdk/releases/download/v1.39.1/operator-sdk_linux_amd64
chmod +x operator-sdk_linux_amd64 && mv operator-sdk_linux_amd64 /usr/local/bin/operator-sdk
```

- init operator helm

```
# 기본 helm template 사용 시
operator-sdk init --plugins helm --domain kw.local --group demo --version v1alpha1 --kind Gitea

# custom chart 를 이용한 설정 시
operator-sdk init --plugins helm --domain kw.local --group demo --version v1alpha1 --kind Gitea --helm-chart gitea-charts/gitea
```

- Gitea template 수정

```
# helm-operators/gitea/helm-charts/gitea/templates/gitea/pvc.yaml
# Go template type 에러로 인해 int로 형변환 13 Line
spec:
  accessModes:
  {{- if gt (int .Values.replicaCount) 1 }}

# helm-operators/gitea/helm-charts/gitea/templates/gitea/config.yaml
# Line 29 replicaCount 형변환

    {{- /* multiple replicas assertions */ -}}
    {{- if gt (int .Values.replicaCount) 1 -}}
      {{- if .Values.gitea.config.cron -}}
        {{- if .Values.gitea.config.cron.GIT_GC_REPOS -}}

```

- 도커 이미지 빌드, 클러스터에 오퍼레이터 배포

```
# IMG 로 Tag 명 지정
IMG=kw.local/gitea-operator:v2 make docker-build

IMG=kw.local/gitea-operator:v2 make docker-build
```

- 클러스터 롤/롤바인딩 및 rbac 확인

```
# config/rbac의 operator 권한 설정을 확인
# 오퍼레이터의 서비스 어카운트에 대한 ClusterRole ClusterRoleBinding에 따라 생성권한 적용이 되어 해당 값이 적절하도록 수정 필요

# role.yaml 수정
# helm-operators/gitea/config/rbac/role.yaml
# 생성된 operator 네임스페이스의 서비스어카운트와 클러스터롤 매핑 확인

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: manager-role
rules:
- apiGroups:
  - "*"
  resources:
  - namespaces
  - networkpolicies
  - poddisruptionbudgets
  - secrets
  - events
  - deployments
  - statefulsets
  - configmaps
  - persistentvolumeclaims
  - serviceaccounts
  - services
  - gitea
  - gitea/status
  - gitea/finalizers 
   verbs:
  - "*"

```

- Gitea 커스텀 리소스 생성

```
# config/crd는 오퍼레이터와 함께 자동생성
cat << EOF >> config/samples/gitea.yaml
apiVersion: demo.kw.local/v1alpha1
kind: Gitea
metadata:
  name: gitea-dev
spec:
  redis-cluster:
    enabled: false
  redis:
    enabled: false
  postgresql:
    enabled: false
  postgresql-ha:
    enabled: false
  
  gitea:
    config:
      database:
        DB_TYPE: sqlite3
      session:
        PROVIDER: memory
      cache:
        ADAPTER: memory
      queue:
        TYPE: level
EOF

kubectl apply -k config/samples/gitea.yaml
```

- 네임스페이스에 gitea 생성확인 

```
> k get gitea.demo.kw.local,pod -n gitea
NAME                            AGE
gitea.demo.kw.local/gitea-dev   21m

NAME                             READY   STATUS    RESTARTS   AGE
pod/gitea-dev-6fd77b9b7d-bsbs5   1/1     Running   0          21m
```