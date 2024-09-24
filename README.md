## 🐋 Docker 이미지 최적화 가이드

### 개요

Docker 이미지를 최적화하는 것은 리소스를 효율적으로 사용하고, 배포 속도를 빠르게 하며, 보안을 강화하는 중요한 작업입니다. 
이 프로젝트는 Docker 컨테이너 최적화 과정을 실습한 프로젝트입니다.

---

### 1. **작은 베이스 이미지 선택하기** 🐧

작은 이미지를 사용하면 빌드와 배포 속도가 빨라집니다.

- **Alpine Linux**: 5MB
- **Distroless**: 2.51MB

``` bash

FROM nginx:alp
```

---

### 2. **단일 책임 원칙 적용하기** 📦

하나의 Docker 이미지는 하나의 작업만 처리하도록 구성합니다. 각 서비스는 별도의 이미지로 관리하면 유연성이 높아집니다.

---

### 3. **멀티 스테이지 빌드 사용하기** 🛠

빌드 단계에서 불필요한 파일을 제외하고, 최종 실행 이미지에 필수적인 파일만 포함시킵니다. 이렇게 하면 이미지 크기가 줄어듭니다.

``` Dockerfile

# 빌드 단계
FROM golang:1.20-alpine AS build
WORKDIR /build
COPY go.mod go.sum ./
RUN go mod download && go mod verify
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o run .

# 실행 단계
FROM gcr.io/distroless/static-debian11
WORKDIR /app
COPY --from=build /build/run .
CMD ["/app/run"]

```
- 결과화면

```bash

username@servername:~/step03Image$ docker images
REPOSITORY               TAG       IMAGE ID       CREATED         SIZE
java-app-optimized       latest    b609fe7b15d8   6 seconds ago   326MB
chinarong2/myimg1        1.0       1bdb19af091b   2 hours ago     471MB

```

---

### 4. **레이어 최소화하기** 🎯

Dockerfile의 각 명령어는 하나의 레이어를 생성합니다. 여러 명령어를 하나로 합쳐 레이어 수를 줄이면 이미지 크기와 빌드 속도가 개선됩니다.

```bash

# Dockerfile.multi-run (여러 레이어로 분리)
FROM ubuntu:20.04

RUN apt-get update
RUN apt-get install -y git
RUN apt-get clean
RUN rm -rf /var/lib/apt/lists/*

```

```bash

# Dockerfile.single-run (레이어 최소화)
FROM ubuntu:20.04

RUN apt-get update && \
    apt-get install -y git && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

```
- 결과 화면 
```bash

username@servername:~/step03Image$ docker images
REPOSITORY               TAG       IMAGE ID       CREATED              SIZE
single-run-image         latest    aaa21d2a4210   10 seconds ago       175MB
multi-run-image          latest    a832425de11d   About a minute ago   229MB

```

---

### 5. **.dockerignore 파일 사용하기** 🗂

빌드 시 불필요한 파일이 이미지에 포함되지 않도록 `.dockerignore` 파일을 설정합니다다.

```dockerignore
node_modules
Dockerfile*
.git
.github
.gitignore
dist/**
README.md

```

---

### 6. **구체적인 이미지 태그 사용하기** 🏷

`latest` 대신 특정 버전을 명시해 일관된 환경을 유지합니다.

```Dockerfile

FROM nginx:1.19

```

---

### 7. **불필요한 패키지 설치 줄이기** ✂

꼭 필요한 패키지만 설치하고, 설치 후 남은 임시 파일은 삭제하여 이미지 크기를 줄이고 보안을 개선할 수 있습니다.

---

## 결론

위의 최적화 방법들을 사용하면 Docker 이미지 크기를 줄이고 빌드 및 배포 속도를 빠르게 할 수 있습니다. 또한 보안 문제도 줄어들어 더 안전한 컨테이너 환경을 만들 수 있습니다. 
프로젝트를 진행하며 Docker 이미지 관리의 중요성을 깨닫고 최적화하는 방법에 대해 배웠습니다!🌟

---

## Reference 🫡
(https://overcast.blog/docker-image-optimization-tips-tricks-6a17f687162b)
<br>
(https://faun.pub/reduce-the-size-of-the-docker-image-e6895b653419)
