# Kubernetes 명령어 정리 — Forus 실습

`kubectl`은 Kubernetes에 리소스 조회·생성·변경 요청을 보내는 명령행 도구입니다.
이 문서는 Forus 로컬 실습 환경을 기준으로 명령어의 의미와 사용 순서를 정리합니다.

## 1. 실습 환경과 기본 문법

| 항목 | 이 실습에서 사용하는 값 |
| --- | --- |
| Kubernetes 접속 설정(Context) | `docker-desktop` |
| Namespace | `forus-local` |
| 애플리케이션 Deployment | `account`, `chat`, `notification` |
| 기타 Deployment | `gateway`, `redis` |
| MySQL StatefulSet / Pod | `mysql` / `mysql-0` |
| 내 PC에서 접속하는 주소 | `http://127.0.0.1:28081` |

다른 환경에서는 Context, Namespace, 리소스 이름을 해당 환경에 맞게 바꿉니다.
Docker Desktop에서 Kubernetes가 실행 중이어야 클러스터에 명령할 수 있습니다.

```bash
kubectl [동작] [대상 종류] [대상 이름] [옵션]
```

예를 들어 다음 명령은 `forus-local` 안에 있는 Pod 목록을 조회합니다.

```bash
kubectl get pods -n forus-local
```

| 부분 | 의미 |
| --- | --- |
| `kubectl` | Kubernetes에 요청하는 도구 |
| `get` | 리소스 조회 |
| `pods` | 조회할 리소스 종류 |
| `-n forus-local` | 요청할 Namespace 지정 |

자주 사용하는 옵션은 다음과 같습니다.

| 옵션 | 의미 |
| --- | --- |
| `--context docker-desktop` | 이 명령에 사용할 접속 설정 지정 |
| `-n forus-local` | Namespace 지정. `--namespace`와 같음 |
| `-A` | 모든 Namespace 대상으로 조회 |
| `-o wide` | IP, 노드 등 추가 정보를 표시 |
| `-o yaml` | 리소스 설정을 YAML로 표시 |
| `-w` | 상태 변화를 계속 관찰 |
| `-f 파일.yaml` | `apply` 등에서 입력 파일 지정 |
| `-f` | `logs`에서는 로그를 계속 출력. 명령에 따라 의미가 다름 |

## 2. 명령을 보낼 클러스터 선택

Context는 클러스터, 인증 사용자, 기본 Namespace를 묶은 접속 설정입니다.

```bash
# 등록된 접속 설정 목록
kubectl config get-contexts

# 현재 선택된 접속 설정
kubectl config current-context

# 기본 접속 설정을 Docker Desktop으로 변경
kubectl config use-context docker-desktop
```

아래 예시는 `docker-desktop`을 선택한 상태를 기준으로 합니다.
기본 선택을 바꾸지 않고 특정 클러스터에만 명령하려면 다음처럼 작성합니다.

```bash
kubectl --context docker-desktop get pods -n forus-local
```

## 3. 실행 상태 조회: get

```bash
# 클러스터의 노드 목록 — 노드는 Namespace에 속하지 않음
kubectl get nodes

# Namespace 목록
kubectl get namespaces

# forus의 Pod 목록
kubectl get pods -n forus-local

# Pod의 IP와 배치된 노드까지 확인
kubectl get pods -n forus-local -o wide

# Pod 상태 변화를 계속 관찰
kubectl get pods -n forus-local -w

# Deployment와 준비된 Pod 수 확인
kubectl get deployments -n forus-local

# MySQL 같은 StatefulSet 확인
kubectl get statefulsets -n forus-local

# Service의 IP와 포트 확인
kubectl get services -n forus-local

# 영속 저장 공간 요청 상태 확인
kubectl get pvc -n forus-local
```

`-w` 명령에서 `Ctrl+C`를 누르면 관찰만 종료됩니다. Pod는 계속 실행됩니다.
Pod 목록의 `READY`는 준비된 컨테이너 수를 나타냅니다. 예를 들어 `1/1`은
표시 대상 컨테이너 1개 중 1개가 준비되었다는 뜻입니다.
Deployment 목록의 `READY`는 준비된 Pod 수 / 원하는 Pod 수입니다.

리소스 이름은 다음처럼 줄여 쓸 수도 있습니다.

| 정식 이름 | 줄임말 |
| --- | --- |
| `pods` | `po` |
| `services` | `svc` |
| `deployments` | `deploy` |
| `statefulsets` | `sts` |
| `namespaces` | `ns` |
| `persistentvolumeclaims` | `pvc` |

## 4. 실행 실패 원인 확인: describe, events

```bash
# account Deployment의 설정과 이벤트
kubectl describe deployment account -n forus-local

# 특정 Pod의 컨테이너 상태, 볼륨, 이벤트
kubectl describe pod POD이름 -n forus-local

# 이벤트를 생성 시각순으로 조회
kubectl get events -n forus-local --sort-by=.metadata.creationTimestamp
```

`POD이름`은 그대로 입력하는 값이 아닙니다.
`kubectl get pods -n forus-local`에 나온 실제 이름으로 바꿉니다.

Pod가 실행되지 않으면 `describe pod`의 컨테이너 상태와 `Events`부터 확인합니다.
이미지 다운로드 실패, 노드 자원 부족, 볼륨 연결 문제, 헬스 체크 실패 등의
원인을 확인할 수 있습니다.

## 5. 프로그램 로그 확인: logs

```bash
# account 로그 최근 100줄
kubectl logs deployment/account -n forus-local --tail=100

# chat 로그를 계속 출력
kubectl logs deployment/chat -n forus-local -f

# notification 로그 최근 100줄
kubectl logs deployment/notification -n forus-local --tail=100

# MySQL 로그
kubectl logs mysql-0 -n forus-local --tail=100

# 특정 Pod에서 재시작하기 전 컨테이너의 로그
kubectl logs POD이름 -n forus-local --previous

# Pod 안에 컨테이너가 여러 개일 때 대상 컨테이너 지정
kubectl logs POD이름 -n forus-local -c 컨테이너이름
```

`--previous`는 같은 Pod에서 이전에 종료된 컨테이너의 로그를 조회합니다.
삭제된 모든 과거 Pod의 로그를 찾아주는 기능은 아닙니다.

Deployment의 Pod가 여러 개면 `logs deployment/account`가 모든 Pod 로그를
합쳐 보여준다고 생각하면 안 됩니다. 특정 Pod 이름을 지정하거나,
현재 Forus의 `app=account` 라벨로 여러 Pod를 선택합니다.

```bash
kubectl logs -n forus-local -l app=account --all-containers=true --prefix=true --tail=100
```

## 6. 내 PC에서 접속하기: port-forward

```bash
kubectl --context docker-desktop --namespace forus-local \
  port-forward --address=127.0.0.1 service/gateway 28081:8080
```

| 부분 | 의미 |
| --- | --- |
| `--address=127.0.0.1` | 내 PC의 loopback 주소에서 연결을 받음 |
| `service/gateway` | gateway Service를 기준으로 대상 Pod를 선택 |
| `28081` | 내 PC에서 열 포트 |
| `8080` | gateway Service의 포트. 대응하는 Pod의 targetPort로 전달 |

```text
브라우저 → 127.0.0.1:28081 → kubectl 포트 포워딩 → 선택된 gateway Pod
```

포트 포워딩을 실행한 터미널을 켜둔 상태에서 다음 주소에 접속합니다.

- Swagger UI: `http://127.0.0.1:28081/swagger-ui/index.html`
- 상태 확인: `http://127.0.0.1:28081/health`

`Ctrl+C`를 누르면 포트 포워딩만 종료되고 Pod는 계속 실행됩니다.
선택된 Pod가 종료되면 명령을 다시 실행해야 합니다.
Service를 지정해도 이 터널이 여러 gateway Pod로 요청을 분산하는 것은 아닙니다.
`127.0.0.1`은 접속하는 기기 자신을 가리키므로 다른 PC에서 이 주소로 접속할 수는 없습니다.

## 7. Pod 개수 변경: scale

다음 명령은 실제 실행 상태를 변경합니다.

```bash
# account Pod를 2개 유지하도록 변경
kubectl scale deployment/account -n forus-local --replicas=2

# 다시 1개로 변경
kubectl scale deployment/account -n forus-local --replicas=1
```

`replicas=2`는 **Pod 두 개를 유지하라**는 요청입니다.
노드가 하나뿐이면 두 Pod 모두 같은 노드에 배치됩니다.
노드가 여러 개여도 별도 배치 조건이 없으면 서로 다른 노드에 배치된다는 보장은 없습니다.

`scale`로 변경해도 로컬 YAML 파일의 `replicas` 값은 수정되지 않습니다.
이후 YAML을 다시 적용하면 YAML에 적힌 값으로 바뀔 수 있습니다.

현재 Forus의 chat은 메모리 내부 Simple Broker를 사용합니다.
chat Pod를 여러 개로 늘리기 전에 서로 다른 Pod에 접속한 사용자 간에도 메시지가
전달되도록 공유 브로커나 이벤트 전파 구조를 준비해야 합니다.

## 8. Pod 교체 요청과 진행 확인: rollout

```bash
# account Deployment의 Pod 교체 요청
kubectl rollout restart deployment/account -n forus-local

# 교체가 완료되는지 기다리기
kubectl rollout status deployment/account -n forus-local
```

`rollout restart`는 현재 Deployment 설정으로 Pod를 교체합니다.
Java 소스를 빌드하거나 새 Docker 이미지를 만들어주지는 않습니다.
교체 순서는 Deployment의 업데이트 전략과 준비 상태에 따라 달라집니다.

## 9. YAML 적용: apply

```bash
# 지정한 YAML의 리소스를 생성하거나 변경
kubectl apply -n forus-local -f 파일.yaml
```

`파일.yaml`은 적용할 실제 파일 경로로 바꿉니다.
YAML에 `metadata.namespace`가 있다면 지정한 Namespace와 일치해야 합니다.

`apply`는 설정을 Kubernetes에 전달하는 단계입니다.
프로그램 변경을 배포할 때는 이미지 빌드와 해당 이미지 태그 지정도 필요합니다.

Forus 로컬 실습용 `scripts/k8s`는 이 과정을 묶은 **Bash 스크립트**입니다.
해당 스크립트가 있는 `forus-msa` 프로젝트 루트에서 실행합니다.

```bash
# 세 서비스 테스트·이미지 빌드 → 설정 적용 → 준비 완료 대기
./scripts/k8s up

# 포트 포워딩 — 이 터미널은 유지
./scripts/k8s open

# 실행 상태 조회
./scripts/k8s status

# account 로그 확인
./scripts/k8s logs account

# 실행 중지 — DB 볼륨과 설정은 보존
./scripts/k8s stop
```

이 명령들은 프로젝트에서 만든 도구의 명령이며, `kubectl` 자체의 하위 명령은 아닙니다.
이 문서만 복사해도 스크립트나 Kubernetes 설정이 함께 설치되는 것은 아닙니다.
`up`은 로컬 구성에 정의된 Pod 수로 다시 적용하므로 수동으로 늘린 개수가 유지되지 않을 수 있습니다.

## 10. 처음 익힐 때의 확인 순서

1. `config current-context`: 내가 어느 클러스터에 명령하고 있는지 확인합니다.
2. `get pods -o wide`: Pod 상태, IP, 노드를 확인합니다.
3. `describe pod`: 준비되지 않은 Pod의 상태와 이벤트를 확인합니다.
4. `logs`: 프로그램이 출력한 오류와 처리 내역을 확인합니다.
5. `port-forward`: 준비된 애플리케이션에 내 PC에서 접속합니다.

## 공식 참고 자료

- [kubectl 명령어 빠른 참고](https://kubernetes.io/docs/reference/kubectl/quick-reference/)
- [Context와 kubeconfig](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)
- [포트 포워딩](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/)
- [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
