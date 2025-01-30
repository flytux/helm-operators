### Helm-Operators

- Operator SDK를 이용하여 helm chart 기반의 CRD를 생성하고 관리한다.
- helm chart에서 정의되는 values 파일의 내용을 CRD에 정의하여 helm chart 기반의 쿠버네티스 객체를 설치할 수 있다.
- operator는 각 하나의 helm chart애 대응되며, 설치하려는 helm chart를 포함하는 operator 이미지와 rbac 설정 bundle로 설치된다.
- helm chart로 설치 관리될 인프라 서비스를 간단한 CRD로 설치할 수 있으며, operator lifecycle 관리가 적용되어 통해 서비스 상태를 선언적으로 관리할 수 있다.
- CRD의 값을 변경하므로서 자동 적용

---

### Operator SDK Command

```
# helm chart를 받아서 해당 폴더에 압축을 해제한 후 Chart.yaml과 values.yaml 값을 적절하게 편집

# helm chart operator 생성
operator-sdk init --plugins helm --domain kw.local --group demo --version v1alpha1 --kind Harbor --helm-chart harbor
IMG=kw.local/harbor-operator:v1 make docker-build
IMG=kw.local/harbor-operator:v1 make deploy

# 이미지 저장 및 노드에 복사
docker save -o harbor-operator.tar kw.local/harbor-operator:v1
scp harbor-operator.tar node-02.local:/root
ssh node-02.local
nerdctl load -i harbor-operator.tar

# CR 생성
k delete -k config/samples/ -n harbor
k apply -k config/samples -n harbor

```

### Sample operators

- Gitea (gitea-charts의 표준 차트 이용)
- Nginx (기본 템플릿 자동 생성 방식)
- Argocd
- Nexus
- Apache (Bitnami Chart의 경우 chart dependency의 경로를 로컬 file:// 로 변경하고, 타 정보를 제거한후에 실행)
- Harbor (Apache와 같이 디펜던시 경로 로컬로 변경, reconsile이 지속되어 jobservice readiness probe 삭제), upgrade 시 registry password를 입력해야 가능하여 해당 로직 처리 필요
