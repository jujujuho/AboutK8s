# Docker

---

## 컨테이너란?

코드와 종속성을 패키지화하여 **다른 환경에서도 안정적으로 실행**할 수 있도록 만드는 기술.  
이를 통해 **Runtime Environment**를 제공합니다.

- 기존 하이퍼바이저 방식의 가상화나 VM보다 **속도가 빠르다**.
- **독립적인 환경**으로 다른 컨테이너에 영향을 주거나 받지 않는다.

---

## Image란?

Docker 이미지란 코드, 라이브러리, 종속성, 도구 등 컨테이너 실행에 필요한 **모든 파일을 포함한 템플릿**이다.  
수정이 불가하며, 보통 **스냅샷**이라고도 불린다.

### Workflow

DockerFile ----build----> Image ----create----> Container

---

## DockerFile이란?

Docker 이미지를 생성하기 위한 지시사항을 **텍스트 파일**로 정의한 것.

- Docker 이미지를 구성하는 과정을 **자동화**함.
- 이미지 빌드 과정을 **반복 가능**하게 만들고, 버전 관리가 가능.

---

## DockerFile 작성 방법

1. **베이스 이미지 선택**: 컨테이너 생성 시, 기존 베이스 이미지를 기반으로 시작.
2. **파일 및 명령 추가**: 필요한 파일 복사, 환경 변수 설정, 애플리케이션 설치 등을 정의.
3. **작업 디렉토리 설정**: 명령이 실행될 위치를 지정.
4. **명령 실행**: RUN, CMD, ENTRYPOINT 등을 사용해 실행할 명령어를 정의.
5. **포트 노출**: EXPOSE 명령어를 통해 컨테이너에서 사용할 포트를 지정.
6. **이미지 메타데이터 설정**: 이미지의 메타데이터를 설정.

---

## DockerFile 명령어 예시

| 명령어         | 설명                                                                                      | 예시                                              |
|----------------|------------------------------------------------------------------------------------------|--------------------------------------------------|
| `FROM`         | 베이스 이미지 지정                                                                       | `FROM ubuntu:20.04`                              |
| `WORKDIR`      | 작업 디렉토리 설정                                                                       | `WORKDIR /usr/src/app`                           |
| `ENV`          | 환경 변수 설정                                                                           | `ENV APP_ENV=production`                         |
| `COPY`         | 파일이나 디렉토리를 이미지 내부로 복사                                                    | `COPY app /usr/src/app`                          |
| `ADD`          | COPY와 유사하나 URL 다운로드 또는 압축 파일 풀기도 지원                                   | `ADD https://example.com/file.txt /app/`         |
| `RUN`          | 컨테이너 내에서 실행할 명령어 정의 (패키지 설치 등)                                       | `RUN apt-get update && apt-get install -y curl` |
| `EXPOSE`       | 컨테이너가 개방할 포트 지정                                                              | `EXPOSE 80`                                      |
| `CMD`          | 컨테이너 실행 시 기본으로 실행할 명령어 (실행 시 명령어가 없을 경우 실행)                 | `CMD ["npm", "start"]`                           |
| `ENTRYPOINT`   | 컨테이너 실행 시 항상 실행할 명령어 (추가 명령어를 인자로 전달 가능)                     | `ENTRYPOINT ["npm", "start"]`                   |
| `VOLUME`       | 호스트와 컨테이너 간 데이터를 공유할 볼륨 지정                                            | `VOLUME /data`                                   |

---

## 이미지 빌드 예시

1. DockerFile 작성:
    ```dockerfile
    FROM ubuntu:20.04
    WORKDIR /usr/src/app
    COPY app /usr/src/app
    RUN apt-get update && apt-get install -y curl
    CMD ["bash"]
    ```

2. 이미지 빌드:
    ```bash
    docker build -t my-image .
    ```

3. 컨테이너 생성 및 실행:
    ```bash
    docker run -it --name my-container my-image
    ```

