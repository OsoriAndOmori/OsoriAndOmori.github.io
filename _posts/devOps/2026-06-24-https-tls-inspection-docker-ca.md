---
title: HTTPS 와 사내 TLS Inspection, Docker CA 삽질 정리
author: OsoriAndOmori
date: 2026-06-24 12:00:00 +0900
categories: [Blogging, DevOps]
tags: [https, tls, docker, ca, alpine, npm]
---

회사망에서 Docker image 를 빌드하다가 `npm ci`, `apk add` 가 인증서 문제로 깨졌다.

처음에는 그냥 "CA 파일 넣으면 되는 거 아닌가?" 정도로 생각했는데, 막상 삽질해보니
내가 HTTPS 와 사내 TLS inspection 을 정확히 이해하지 못하고 있었다.

그래서 쉬운 말로 다시 정리해본다.

### 일반적인 HTTPS

원래 HTTPS 는 대충 이렇게 동작한다.

```text
내 프로그램
  -> https://registry.npmjs.org 접속
  -> registry.npmjs.org 서버가 인증서를 보여줌
  -> 내 프로그램이 "이 인증서를 믿어도 되나?" 확인
  -> 믿을 수 있으면 암호화 통신 시작
```

인증서에는 보통 이런 정보가 들어있다.

```text
이 인증서는 registry.npmjs.org 용입니다.
이 인증서는 공인 CA 가 서명했습니다.
유효기간은 언제까지입니다.
```

내 PC, 브라우저, 컨테이너 안에는 기본적으로 믿을 수 있는 공인 CA 목록이 들어있다.

```text
DigiCert
GlobalSign
Let's Encrypt
Amazon Trust Services
...
```

그래서 외부 사이트 인증서가 이런 공인 CA 에 의해 서명되어 있으면 "오케이, 믿을 수
있다" 하고 통신한다.

### 사내 TLS Inspection 이 끼면

회사망에서는 보안장비가 HTTPS 트래픽을 검사하는 경우가 있다. 흔히 TLS inspection,
SSL inspection 이라고 부른다.

보안장비가 없으면 그림이 단순하다.

```text
내 프로그램  <========== 암호화 통신 ==========>  외부 사이트
```

보안장비가 있으면 HTTPS 연결이 두 개로 쪼개진다.

```text
내 프로그램  <==== 암호화 ====>  사내 보안장비  <==== 암호화 ====>  외부 사이트
```

내 프로그램 입장에서는 외부 사이트와 직접 TLS 를 맺는 게 아니라, 사내 보안장비와 TLS
를 맺게 된다.

그런데 내 프로그램은 여전히 이렇게 요청한다.

```text
https://registry.npmjs.org
```

그러면 사내 보안장비는 내 프로그램에게 이런 인증서를 보여준다.

```text
이 인증서는 registry.npmjs.org 용입니다.
이 인증서는 사내 Root CA 가 서명했습니다.
```

이게 내가 헷갈렸던 "재서명"이다.

원래 외부 사이트 인증서는 이런 모양이다.

```text
registry.npmjs.org 인증서
서명자: 공인 CA
```

사내 보안장비가 내 프로그램에게 보여주는 인증서는 이런 모양이다.

```text
registry.npmjs.org 인증서처럼 생긴 새 인증서
서명자: 사내 Root CA
```

도메인 이름은 `registry.npmjs.org` 로 맞춰서 새로 만들어주지만, 서명자가 공인 CA 가
아니라 사내 CA 인 것이다.

그래서 내 프로그램이 사내 CA 를 모르면 이렇게 판단한다.

```text
도메인은 registry.npmjs.org 가 맞긴 한데,
이걸 서명한 CA 를 나는 모르겠는데?
못 믿겠음.
```

그 결과 이런 에러가 난다.

```text
SELF_SIGNED_CERT_IN_CHAIN
certificate verify failed
server certificate not trusted
unable to get local issuer certificate
```

반대로 사내 CA 를 trust store 에 넣어두면 이렇게 된다.

```text
이 인증서는 사내 Root CA 가 서명했네.
내 trust store 에 사내 Root CA 가 있네.
그럼 믿을 수 있음.
```

### 왜 내 PC 에서는 되는데 Docker build 에서는 안 되나

핵심은 trust store 가 서로 다르다는 것이다.

```text
내 PC
  └─ 사내 CA 설치되어 있음

GitLab Runner pod
  └─ 사내 CA 설치되어 있을 수 있음

docker build 안의 builder image
  └─ 깨끗한 node:alpine 이미지
  └─ 사내 CA 모름
```

Dockerfile 에서 이런 걸 쓴다고 해보자.

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app

COPY package*.json ./
RUN npm ci
```

`RUN npm ci` 는 내 PC 에서 실행되는 게 아니다. `node:20-alpine` 이라는 별도 이미지
파일시스템 안에서 실행된다.

그래서 내 PC 나 Runner 에 사내 CA 가 설치되어 있어도, builder image 안의 Node/npm 은
그 CA 를 모른다.

결국 `npm ci` 가 npm registry 에 붙을 때 이런 식으로 깨진다.

```text
npm error SELF_SIGNED_CERT_IN_CHAIN
npm error request to https://registry.npmjs.org/... failed
```

### Node/npm 에 CA 를 알려주는 방법

빌드 컨텍스트에 사내 CA 파일을 넣어둘 수 있다면, builder stage 에 복사해서 Node/npm
이 보게 만들 수 있다.

여기서는 공개 문서용으로 파일명을 `internal-ca.pem` 이라고 적는다.

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app

COPY ca-bundle/internal-ca.pem /usr/local/share/ca-certificates/internal-ca.crt

ENV NODE_EXTRA_CA_CERTS=/usr/local/share/ca-certificates/internal-ca.crt \
    npm_config_cafile=/usr/local/share/ca-certificates/internal-ca.crt

COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build
```

여기서 중요한 점은 이 설정이 OS 전체 trust store 를 고친 것은 아니라는 것이다.

```text
NODE_EXTRA_CA_CERTS -> Node.js TLS 에 추가 CA 를 알려줌
npm_config_cafile  -> npm 이 사용할 CA 파일을 알려줌
```

즉 `npm ci` 에는 효과가 있지만, 모든 Linux 프로그램이 자동으로 이 CA 를 믿는다는 뜻은
아니다.

### Alpine 의 apk add 는 왜 또 깨지나

처음에는 OS trust store 에도 CA 를 넣고 싶어서 이렇게 했다.

```dockerfile
COPY ca-bundle/internal-ca.pem /usr/local/share/ca-certificates/internal-ca.crt

RUN apk add --no-cache ca-certificates && \
    update-ca-certificates
```

그런데 이게 Alpine 에서는 먼저 깨질 수 있다.

```text
fetching https://dl-cdn.alpinelinux.org/alpine/.../APKINDEX.tar.gz:
TLS: server certificate not trusted
ERROR: unable to select packages:
  ca-certificates (no such package)
```

`ca-certificates` 패키지가 진짜 없는 게 아니다.

문제는 `apk add` 가 먼저 Alpine repository 에 HTTPS 로 접속해야 한다는 점이다. 그런데
그 순간 아직 사내 CA 는 OS trust store 에 들어가지 않았다.

순서가 이렇게 꼬인다.

```text
Alpine HTTPS repository 를 믿으려면 사내 CA 가 필요
사내 CA 를 OS trust store 에 넣으려면 update-ca-certificates 가 필요
update-ca-certificates 를 쓰려면 ca-certificates 패키지가 필요
ca-certificates 를 설치하려면 먼저 Alpine repository 에 접속해야 함
```

이런 순환 의존 문제가 생긴다.

### Alpine 에서 우회한 방법

단기적으로는 Alpine repository 를 일단 HTTP 로 바꿔서 `ca-certificates` 설치 단계의
TLS 검증을 피할 수 있다.

```dockerfile
COPY ca-bundle/internal-ca.pem /usr/local/share/ca-certificates/internal-ca.crt

RUN sed -i 's|https://dl-cdn.alpinelinux.org|http://dl-cdn.alpinelinux.org|g' /etc/apk/repositories && \
    apk add --no-cache ca-certificates && \
    update-ca-certificates
```

이렇게 하면 일단 `apk add ca-certificates` 를 할 수 있고, 이후
`update-ca-certificates` 가 `/usr/local/share/ca-certificates/internal-ca.crt` 를
OS CA bundle 에 합쳐준다.

그 뒤에는 OS trust store 를 보는 도구들이 사내 TLS inspection 인증서를 믿을 수 있다.

```text
curl
wget
openssl
nginx 의 https upstream
일부 Python/Go/native binary
```

다만 이건 어디까지나 단기 우회다. HTTP repository 로 바꾸면 패키지 전송 구간이
암호화되지 않는다. Alpine 패키지 서명 검증은 남아 있지만, 기분 좋은 최종 형태는
아니다.

장기적으로는 이런 방식이 낫다.

```text
사내 CA 가 이미 포함된 표준 base image 사용
사내 artifact proxy 사용
사내 Alpine mirror 사용
가능하면 Debian slim 같은 이미지로 통일
```

### Debian 에서는 왜 운 좋게 됐나

Debian 계열 이미지에서는 이런 명령이 잘 됐다.

```dockerfile
COPY ca-bundle/internal-ca.pem /usr/local/share/ca-certificates/internal-ca.crt

RUN apt-get update && \
    apt-get install -y --no-install-recommends ca-certificates && \
    update-ca-certificates
```

이유는 단순했다.

내가 쓰던 Debian 이미지의 apt repository 가 HTTP 로 설정되어 있었다.

```text
http://deb.debian.org/debian
```

그래서 `apt-get update` 단계에서 TLS 검증을 하지 않았고, `ca-certificates` 설치까지
갈 수 있었다.

반면 Alpine 은 기본 repository 가 HTTPS 였다.

```text
https://dl-cdn.alpinelinux.org/alpine
```

즉 Debian 이 더 똑똑해서 된 게 아니라, HTTP repository 라서 이 문제를 안 밟았던 것에
가깝다.

### 환경변수는 꼭 필요한가

OS trust store 에 CA 를 제대로 등록했다면 많은 도구는 그냥 동작한다.

그렇다면 이런 환경변수는 필요 없을 수도 있다.

```dockerfile
ENV NODE_EXTRA_CA_CERTS=/usr/local/share/ca-certificates/internal-ca.crt \
    npm_config_cafile=/etc/ssl/certs/ca-certificates.crt \
    SSL_CERT_FILE=/etc/ssl/certs/ca-certificates.crt
```

하지만 Node/npm 은 기본적으로 Node 내부 CA bundle 을 쓰는 경우가 있어서, OS trust
store 만으로 충분하지 않을 때가 있다.

그래서 Node/npm 빌드에서는 보험처럼 같이 두는 편이 안전하다.

```text
update-ca-certificates -> OS trust store 처리
NODE_EXTRA_CA_CERTS    -> Node.js 용 안전장치
npm_config_cafile      -> npm 용 안전장치
SSL_CERT_FILE          -> OpenSSL 계열 도구용 힌트
```

### 정리

이번 삽질로 이해한 건 이렇다.

- HTTPS 는 서버 인증서를 믿을 수 있는 CA 가 서명했는지 확인한다.
- 사내 TLS inspection 은 중간에서 인증서를 새로 만들어 사내 CA 로 재서명한다.
- 그래서 컨테이너 안에도 사내 CA 를 넣어야 한다.
- 내 PC, Runner, Docker builder image 의 trust store 는 전부 별개다.
- `NODE_EXTRA_CA_CERTS`, `npm_config_cafile` 은 Node/npm 에 대한 해결이다.
- `update-ca-certificates` 는 OS trust store 에 대한 해결이다.
- Alpine 은 `apk add ca-certificates` 자체가 HTTPS repository 신뢰 문제에 막힐 수 있다.
- Debian 에서 됐던 건 apt repository 가 HTTP 라서 운 좋게 부트스트랩된 것일 수 있다.

결국 "CA 파일을 넣었다"가 중요한 게 아니라, **어떤 프로세스가 어떤 trust store 를 보고
있는지**를 알아야 한다.

이걸 모르면 같은 CA 파일을 들고도 `npm`, `apk`, `curl`, `nginx`, `Python` 이 서로 다른
방식으로 터진다.
