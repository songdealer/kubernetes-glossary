# 쿠버네티스 용어와 명령어 정리

Kubernetes를 학습하면서 자주 만나는 용어와 명령어를 정리합니다.
명령어 예시는 Forus 로컬 실습 환경을 기준으로 작성했습니다.

## 문서

- [Kubernetes 명령어 정리와 실습 예제](kubernetes-commands.md)

## 기본 용어

| 용어 | 의미 | Forus 실습 예시 |
| --- | --- | --- |
| 클러스터(Cluster) | Kubernetes가 함께 관리하는 노드와 제어 시스템의 집합 | Docker Desktop의 로컬 클러스터 |
| 노드(Node) | Pod를 실행하는 물리 서버 또는 가상 머신 | 로컬 클러스터의 노드 |
| 이미지(Image) | 애플리케이션과 실행에 필요한 파일을 담은 배포 단위 | account-service를 빌드한 이미지 |
| 컨테이너(Container) | 이미지를 바탕으로 격리된 환경에서 실행되는 프로세스 | account-service Java 프로세스 |
| Pod | Kubernetes의 기본 배포 단위. 하나 이상의 컨테이너가 네트워크를 공유하며 같은 노드에서 실행됨 | account 애플리케이션 Pod |
| Deployment | 원하는 Pod 개수를 유지하고 Pod의 버전 교체를 관리하는 리소스 | `account`, `chat`, `notification` |
| StatefulSet | Pod마다 안정된 이름과 영속 볼륨 연결이 필요한 워크로드를 관리하는 리소스 | MySQL의 `mysql-0` Pod |
| Service | Pod가 바뀌어도 일정한 이름과 접속 경로를 제공하는 리소스. 일반적인 구성에서는 준비된 대상 Pod들로 연결을 분산 | `account`, `chat`, `gateway` |
| Namespace | 한 클러스터 안에서 리소스를 이름과 범위로 구분하는 공간 | `forus-local` |
| Context | kubectl이 사용할 클러스터, 인증 사용자, 기본 Namespace를 묶은 접속 설정 | `docker-desktop` |
| Label / Selector | 리소스에 붙이는 식별 정보와 그 정보를 기준으로 대상을 고르는 조건 | `app=account` |
| ConfigMap | 비밀이 아닌 설정 데이터를 보관하는 리소스 | Nginx 설정 |
| Secret | 비밀번호·토큰 등의 민감한 설정을 보관하는 리소스. 접근 제어와 저장 시 암호화 설정도 별도로 중요함 | DB 비밀번호, JWT 키 |
| PVC | Pod가 사용할 영속 저장 공간을 요청하는 리소스 | MySQL 데이터 저장 공간 |
| Replica | 같은 설정으로 실행하는 Pod의 복제본 | account Pod 2개 유지 |
| Port forwarding | 내 PC의 포트를 클러스터 안의 Pod로 임시 연결하는 기능 | `127.0.0.1:28081`에서 gateway 접속 |
| Readiness probe | 컨테이너가 요청을 받을 준비가 되었는지 확인하는 검사 | 준비된 애플리케이션으로 트래픽 전달 |
| Liveness probe | 컨테이너가 정상 동작하는지 확인하고, 설정된 실패 기준을 넘으면 재시작하게 하는 검사 | 응답 불능 애플리케이션 복구 |
| Startup probe | 컨테이너 시작 완료를 확인하는 검사. 성공 전에는 readiness/liveness 검사를 실행하지 않음 | Java 애플리케이션 초기 기동 확인 |

## 학습 순서

1. 노드, Pod, 컨테이너의 관계를 이해합니다.
2. Deployment가 Pod 개수를 관리하고 Service가 접속 경로를 제공하는 흐름을 확인합니다.
3. `kubectl get`, `describe`, `logs`로 실행 상태를 확인합니다.
4. `port-forward`로 직접 접속합니다.
5. `scale`, `rollout`, `apply`가 무엇을 변경하는지 익힙니다.

이 저장소에는 학습 문서만 포함됩니다. 예시에 등장하는 Forus 애플리케이션,
`scripts/k8s` 스크립트와 클러스터 설정은 별도로 준비해야 합니다.

## 공식 문서

- [Kubernetes 개념](https://kubernetes.io/docs/concepts/)
- [Pod](https://kubernetes.io/docs/concepts/workloads/pods/)
- [Service](https://kubernetes.io/docs/concepts/services-networking/service/)
- [kubectl 명령어 참고](https://kubernetes.io/docs/reference/kubectl/quick-reference/)
