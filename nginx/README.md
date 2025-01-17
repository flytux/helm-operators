## Operator SDK - helm chart - Bitnami/Nginx 생성

- download operator-sdk

```
curl -LO https://github.com/operator-framework/operator-sdk/releases/download/v1.39.1/operator-sdk_linux_amd64
chmod +x operator-sdk_linux_amd64 && mv operator-sdk_linux_amd64 /usr/local/bin/operator-sdk
```

- init operator helm

```
# 기본 helm template 사용 시
operator-sdk init --plugins helm --domain kw.local --group demo --version v1alpha1 --kind Nginx

# custom chart 를 이용한 설정 시
operator-sdk init --plugins helm --domain example.com --group demo --version v1alpha1 --kind Nginx --helm-chart bitnami/nginx
```

- 도커 이미지 빌드, 클러스터에 오퍼레이터 배포

```
make docker-build

make deploy
```

- nginx 커스텀 리소스 생성

```
# config/crd는 오퍼레이터와 함께 자동생성
# nginx crd는 해당 chart의 values 파일을 기준으로 생성

kubectl apply -k config/samples

# config/rbac의 operator 권한 설정을 확인
# 오퍼레이터의 서비스 어카운트에 대한 ClusterRole ClusterRoleBinding에 따라 생성권한 적용이 되어 해당 값이 적절하도록 수정 필요
```
