Docker
=========

컨테이너란?
--------

코드와 종속성을 패키지화 하여 다른 PC에서도 안정적으로 실행하게함 --> Runtime Env

1. 기존의 하이퍼바이저식 가상화 방법이나, Vm보다 속도가 빠름
2. 독립적인 환경으로 다른 것에 영향을 받지 않음

![image](https://github.com/user-attachments/assets/aaa7a96c-e5f8-4c20-b59c-02a7317020d1)


Image란?
-------

코드나 라이브러리, 종속성, 도구 등 필요한 기타 파일을 가지는 파일 ( 템플릿 ) 
스냅샷이라고도 불리며, 수정이 불가함.

DockerFile ----build----> Image ----create---> Container 


![image](https://github.com/user-attachments/assets/f71e4222-b704-4249-aa04-6573e02a5487)


DockerFile
----------

-DockerFile이란, 도커 이미지를 생성하기 위한 지시사항들을 텍스트 파일로 만든 것이다. 
-도커이미지를 구성하는 과정을 자동화하고, 이미지가 어떻게 빌드되어야 하는지 세부 사항을 정의할 수 있다. ( 모든 단계 정의 )
-DockerFile을 사용하면 이미지 빌드를 반복 가능하게 하고, 버전 관리 시스템을 통해 이미지 구성을 관리하기 편하다.

# 작성 방법
1. 베이스 이미지 선택: 컨테이너 이미지 구성 시, 기존 이미지( 베이스 이미지 ) 를 기반으로 시작함
2. 필요한 파일 및 명령 추가: 베이스 이미지 위에 필요한 파일과 명령 추가(파일 복사, 환경 변수 설정, 응용 프로그램 설치)
3. 작업 디렉토리 설정: 이미지 내에서 작업 디렉토리를 설정하여 파일을 추가하거나 명령을 실행하는 위치 설정
4. 명령 실행: 이미지 내에서 실행할 명령 정의. Run, CMD, ENTRYPOINT 등
5. 포트 노출: 이미지가 실행될 때, 개방할 포트 지정. EXPOSE 명령을 사용하여 컨테이너 내부에서 개방할 포트를 명시
6. 이미지 메타데이터 설정: 이미지의 메타데이터를 설정할 수 있음

## Ex)
FROM	베이스 이미지 지정	FROM ubuntu:20.04


WORKDIR	작업 디렉토리 설정	WORKDIR /usr/src/app


ENV	환경 변수 설정	COPY app /usr/src/app


COPY	호스트 파일 시스템에서 파일이나 디렉토리를 이미지 내부로 복사	COPY app /usr/src/app


ADD	COPY와 유사하지만 추가 기능을 지원. URL로부터 파일을 다운로드하거나 압축 파일을 풀 수 있음	ADD https://example.com/file.txt /app/


RUN	컨테이너 내에서 실행할 명령어 정의. 패키지 설치, 파일 복사, 응용프로그램 빌드 등	RUN apt-get update && apt-get install -y curl


EXPOSE	컨테이너가 개방할 포트 지정	EXPOSE 80
CMD	컨테이너가 실행될 때 실행할 명령어 지정. `docker run`  명령에서 명령어가 제공되지 않을 경우에만 실행	CMD ["npm", "start"]


ENTRYPOINT	컨테이너가 실행될 때 항상 실행할 명령어. `docker run` 명령에서 추가적인 명령어를 전달하면 해당 명령어가 인자로 사용	ENTRYPOINT ["npm", "start"]


VOLUME	호스트와 컨테이너 간 볼륨을 공유	VOLUME /data
