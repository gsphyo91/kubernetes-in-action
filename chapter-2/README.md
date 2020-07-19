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
