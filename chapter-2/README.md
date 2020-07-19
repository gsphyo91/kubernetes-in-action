# Docker를 사용한 컨테이너 이미지 생성, 실행, 공유

## Docker 설치와 Hello World 컨테이너 실행

### docker 설치

[http://docs.docker.com/engine/installation](http://docs.docker.com/engine/installation)

### Hello World 컨테이너 실행

```bash
# Hello World
docker run busybox echo "Hello world"
```

busybox는 echo, ls, gzip 등과 같은 표준 UNIX 명령줄 도구들을 합쳐 놓은 단일 실행파일(executable)

### 백그라운드에 일어난 동작 이해하기

`docker run` 명령을 수행했을 때, 일어나는 일

1. `docker run busybox echo "Helloworld"`
2. 로컬 머신에 busybox 이미지가 저장돼 있는지 확인
3. 로컬에 이미지가 없으면 도커는 레지스트리로부터 이미지를 pull
4. docker가 격리된 컨테이너에서 echo "Hello World"를 실행

### 컨테이너 이미지에 버전 지정하기

```bash
docker run <image>:<tag>
```

## Node.js 어플리케이션 생성하기

[Source Code](app.js)

## 이미지를 위한 Dockerfile 생성

[Dockerfile](Dockerfile)

Dockerfile에는 docker가 이미지를 생성하기 위해 수행해야 할 지시 사강이 담겨있다.

## 컨테이너 이미지 생성

```bash

docker build -t kubia .
```

### 어떻게 이미지가 빌드되는 지 이해하기

빌드 프로세스는 도커 클라이언트가 수행하지 않는다. 그 대신 디렉토리의 전체 컨텐츠가 도커 데몬에 업로드되고, 그 곳에서 이미지가 빌드된다.

### 이미지 레이터 이해하기

동일한 기본 이미지(node:7)를 바탕으로 다수의 이미지를 생성하더라도 기본 이미지를 구성하는 모든 레이어는 단 한 번만 저장될 것이다. 또한 이미지를 가져올 때도 도커는 각 레이어를 개별적으로 다운로드 한다. 컴퓨터에 여러 개의 레이어가 이미 저장돼 있다면 도커는 저장되지 않은 레이어만 다운로드 한다.

이미지 빌드 프로세스가 완료되면 새로운 이미지가 로컬에 저장된다.

```bash
# 로컬에 저장된 이미지 리스트 조회
docker images
```

## 컨테이너 이미지 실행

```bash
docker run --name kubia-container -p 8080:8080 -d kubia
# --name : kubia 이미지에서 kubia-container라는 이름의 새로운 컨테이너를 실행
# -d : 백그라운드에서 실행
# -p : 로컬 머신의 포트(8080)가 컨테이너 내부의 포트(8080)와 매핑
```

### 어플리케이션 접근

```bash
curl lobalhost:8080
You've hit <Host Name>
# Host Name : 16진수의 docker container ID
```

### 실행 중인 모든 컨테이너 조회

```bash
# 실행 중인 컨테이너 목록
docker ps

# 컨테이너의 자세한 정보 조회
docker inspect kubia-container
```

## 실행 중인 컨테이너 내부 탐색하기

이미지 내에 실행 가능한 셸 바이너리가 제공된다면 셸을 실행할 수 있다.

### 실행 중인 컨테이너 내부에서 셸 실행하기

```bash
docker exec -it kubia-container bash
# -i : 표준 입력(STDIN)을 오픈 상태로 유지한다. 셸에 명령어를 입력하기 위해 필요하다.
# -t : 의사(pseudo) 터미널(TTY)을 할당한다.
```

옵션을 빼면 명령어를 입력할 수 없고, 명령어 프롬프트가 화면에 표시되지 않는다.

## 컨테이너 중지와 삭제

```bash
# 컨테이너 메인 프로세스 중지
docker stop kubia-container

# 컨테이너 완전히 삭제
docker rm kubia-container
```

## 이미지 레지스트리에 이미지 푸시

다른 컴퓨터에서도 이미지를 실행하려면 외부 이미지 저장소에 이미지를 푸시해야 한다.

이미지를 푸시하기 전에 도커 허브의 규칙에 따라 이미지 태그를 다시 지정해야 한다.
도커 허브는 이미지의 레포지토리 이름이 도커 허브 ID로 시작해야만 이미지를 푸시할 수 있다.

### 추가 태그로 이미지 태그 지정

```bash
# 태그 생성
docker tag kubia gsphyo91/kubia
```

이 명령은 태그를 변경하기 않고, 같은 이미지에 추가적인 태그를 생성한다.

### 도커 허브에 이미지 푸시하기

```bash
docker push gsphyo91/kubia
```

### 다른 머신에서 이미지 실행하기

```bash
docker run -d -p 8080:8080 gsphyo91/kubia
```

# 쿠버네티스 클러스터 설치

## Minikube를 활용한 단일 노드 쿠버네티스 클러스터 실행하기

Minikube는 로컬에서 쿠버네티스를 테슽하고, 어플리케이션을 개발하는 목적으로 단일 노드 클러스터를 설치하는 도구이다.

### Minikube 설치

[설치 방법](http://github.com/kubernetes/minikube)

```bash
# MacOS
brew install minikube

# OR
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
sudo install minikube-darwin-amd64 /usr/local/bin/minikube
```

### Minikube로 쿠버네티스 클러스터 시작하기

```bash
# Minikube 가상머신 시작하기
minikube start
```

### Kubernetes 클라이언트 시작하기

```bash
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl"
```

### 클러스터 작동 여부 확인과 kubectl로 사용하기

```bash
# 클러스터 정보 표시하기
kubectl cluster-info
```

### 클러스터의 개념 이해하기

각 노드는 Docker, Kubelet, kube-proxy를 실행한다. `kubectl` 클라이언트 명령어는 마스터 노드에서 실행 중인 쿠버네티스 API 서버로 REST 요청을 보내 클러스터와 상호작용한다.

### 클러스터 노드를 조회해 클러스터 동작 상태 확인하기

```bash
# kubectl로 클러스터 노드 조회하기
kubectl get nodes
```

### 오브젝트 세부 정보 가져오기

```bash
kubectl describe node <node-name>
```

## kubectl의 alias와 명령줄 자동완성 설정하기

### alias 설정하기

```bash
alias k=kubectl
```

# kubernetes에 첫 번째 어플리케이션 실행하기

## Node.js 어플리케이션 구동하기

어플리케이션을 배포하기 위한 가장 간단한 방법은 `kubectl ren` 명령어를 사용하여 JSON이나 YAML을 사용하지 않고 필요한 모든 구성요소를 생성하는 방법이다.

```bash
# 이전에 생성해서 Docker hub에 push한 이미지를 실행
kubectl run kubia --image=gsphyo91.kubia --port=8080 --generator=run/v1
```

- `--image=gsphyo91/kubia`: 실행하고자 하는 컨테이너 이미지를 명시
- `-port=8080`: kubernetes에 어플리케이션이 8080포트를 수신 대기해야 한다.
- `--generator`
  - kubernetes에서 deployment 대신 replicationController를 생성하기 때문에 사용

### Pod 소개

컨테이너의 그룹을 파드(Pod)라고 한다.

파드는 하나 이상의 밀접하게 연관된 컨테이너의 그룹으로 같은 워커 노드에서 같은 리눅스 네임스페이스로 함께 실행된다. 각 파드는 자체 IP, 호스트 이름, 프로세스 등이 있는 논리적으로 분리된 머신이다. 파드에서 실행 중인 모든 컨테이너는 동일한 논리적인 머신에서 실행하는 것처럼 보이는 반면, 다른 파드에 실행 중인 컨테이너는 같은 워커노드에서 실행 중이라 할지라도 다른 머신에서 실행 중인 것으로 나타난다.

### Pod 조회하기

```bash
# pod 조회
kubectl get pods

# pod 상세 정보 조회
kubectl describe pod
```

### 백그라운드에 일어난 동작 이해하기

1. `docker push gsphyo91/kubia` : gsphyo91/kubia 이미지를 docker hub에 푸시
2. `kubectl run kubia --image=gsphyo91/kubia --port=8080`
3. kuectl은 REST 호출을 한다.
4. 파드가 생성되고 워커 노드에 스케줄링 된다.
5. kubelet은 통지를 받는다.
6. kubelet은 도커에 이미지를 실행하라고 지시한다.
7. 도커는 이미지를 pull하고, gsphyo91/kubia를 실행한다.
